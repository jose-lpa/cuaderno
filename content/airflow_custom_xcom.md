Title: Backend personalizado de XCom para Airflow
Date: 2023-03-09 17:23
Category: Programación
Tags: airflow,data,xcom,python
Slug: airflow-custom-xcom
Authors: José L. Patiño-Andrés
Summary: Implementación de un backend XCom personalizado para Apache Airflow.

Ésto fue lo que hice para implementar un backend de XCom para Apache Airflow que
guarda los datos en un bucket de Google Cloud Storage.

La implementación es flexible, de manera que podemos especificar el nombre del
bucket en una variable de Airflow `XCOM_BACKEND_DATA_GCS_NAME` y así desplegar
múltiples instancias de Airflow o de Cloud Composer, cada una con su bucket
definido en sí misma.

Como éste ejemplo se trata de una instancia de Airflow en GCP Cloud Composer,
la conexión al bucket mediante IAM ya está definida por el propio Cloud Composer
y por lo tanto no tenemos que hacer nada más que especificarla en el `GCSHook`:
`google_cloud_storage_default`. Si estuviéramos ejecutando Airflow en algo que
no fuera GCP, deberíamos arreglar ésto aparte y crear un usuario IAM para que
Airflow pudiera acceder al bucket.

La idea se sacó de ésta entrada de blog de [Astronomer](https://docs.astronomer.io/learn/xcom-backend-tutorial).

    :::python
    import json
    import tempfile
    import uuid
    from functools import lru_cache
    from typing import Any, Optional

    from airflow.models import Variable
    from airflow.models.xcom import BaseXCom
    from airflow.providers.google.cloud.hooks.gcs import GCSHook


    @lru_cache(maxsize=1)
    def get_xcom_backend_bucket():
        """Helper function to get the name of the GCS bucket used for XCom bucket from the Airflow
        variables.
        """
        return Variable.get("XCOM_BACKEND_DATA_GCS_NAME")


    class GCPBucketXComBackend(BaseXCom):
        # The prefix is optional and used to make it easier to recognize which reference strings in the
        # Airflow metadata database refer to an XCom that has been stored in a GCS bucket.
        prefix = "gcs://"

        @staticmethod
        def serialize_value(
            value: Any,
            *,
            key: Optional[str] = None,
            task_id: Optional[str] = None,
            dag_id: Optional[str] = None,
            run_id: Optional[str] = None,
            map_index: Optional[int] = None,
        ) -> str:
            # Connection to GCS is created by using the GCShook with the connection ID configured by GCP
            # Cloud Composer, which is `google_cloud_storage_default`. Check Airflow Connections screen
            # in Airflow UI to see it.
            hook = GCSHook(gcp_conn_id="google_cloud_storage_default")
            # Create a unique file ID to store XCom data in the GCS bucket.
            gcs_key = f"{run_id}/{task_id}/{uuid.uuid4()}.json"

            # Write the value to a local temporary JSON file.
            with tempfile.NamedTemporaryFile(mode="a+", encoding="utf-8") as tmp_file:
                json.dump(value, tmp_file)

                # (!) Rewind - move the cursor again to the beginning of the file.
                tmp_file.seek(0)

                # Load the local JSON file into the GCS bucket.
                hook.upload(
                    filename=tmp_file.name,
                    object_name=gcs_key,
                    bucket_name=get_xcom_backend_bucket(),
                )

            # Define the string that will be saved to the Airflow metadata database to refer to this
            # XCom.
            reference_string = f"{GCPBucketXComBackend.prefix}{gcs_key}"

            # Use JSON serialization to write the reference string to the Airflow metadata database
            # (like a regular XCom).
            return BaseXCom.serialize_value(value=reference_string)

        @staticmethod
        def deserialize_value(result: BaseXCom) -> str:
            # Retrieve the relevant reference string from the metadata database.
            reference_string = BaseXCom.deserialize_value(result=result)

            # Create the GCS connection using the GCSHook and recreate the key.
            hook = GCSHook(gcp_conn_id="google_cloud_storage_default")
            gs_key = reference_string.replace(GCPBucketXComBackend.prefix, "")

            # Download the JSON file found at the location described by the reference string. Load it
            # into a temporary local file.
            with tempfile.NamedTemporaryFile(mode="wb+") as tmp_file:
                tmp_file.write(
                    hook.download(
                        object_name=gs_key,
                        bucket_name=get_xcom_backend_bucket(),
                    )
                )

                # (!) Rewind - move the cursor again to the beginning of the file.
                tmp_file.seek(0)

                # Return the data as a Python data structure to be used by the operator.
                return json.load(tmp_file)

