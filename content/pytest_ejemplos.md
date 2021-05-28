Title: Ejemplos de uso de Pytest
Date: 2021-05-26 11:33
Category: Programación
Tags: python,pytest,testing
Slug: pytest-ejemplos
Authors: José L. Patiño-Andrés
Summary: Algunos ejemplos de casos de uso de Pytest.

A continuación se muestran algunos ejemplos de tests escritos con la librería
[Pytest](https://pytest.org). Son casos de uso especiales que resuelven algunas
situaciones que pueden darse al escribir tests en proyectos Python.

## Usar fixtures sin necesidad de pasarlas como argumentos

Normalmente, podemos pasar las fixtures que hayamos creado para nuestros tests
como argumentos a las propias funciones de tests:

En `conftest.py`:

    :::python
    from unitest.mock import patch

    import pytest


    @pytest.fixture
    def mock_api_call():
        with patch("project.api_client.get_products") as mock_products:
            mock_products.return_value = ["Product-1", "Product-2", "Product-3]
            yield

En `test_products.py`:

    :::python
    def test_get_products(mock_api_call):
        assert my_module.list_products == ["Product-1", "Product-2", "Product-3]

Esto es muy conveniente, pero presenta un problema cuando usamos linters en
nuestro proyecto, como por ejemplo Pylint. Pylint devuelve un error en este 
código:

`W0613: Unused argument 'mock_api_call' (unused-argument)`

Esto lo podemos arreglar marcando las fixtures de Pytest con el decorador
adecuado, de manera que no necesitaremos pasar argumentos (además de quedar más
claro lo que se está haciendo, para gente no familiarizada con Pytest):

    :::python
    import pytest


    @pytest.mark.usefixtures("mock_api_call)
    def test_get_products():
        assert my_module.list_products == ["Product-1", "Product-2", "Product-3]

De esta forma, los linters no devolverán más errores.

## Asignar nombre a las funciones de fixture

Siguiendo con el ejemplo anterior, se podría dar el caso de que necesitásemos
usar la función `mock_api_call` en otra fixture. Como Pytest permite encadenar
fixtures, esto es sencillo de hacer:

    :::python
    @pytest.fixture
    def mock_api_call():
        with patch("project.api_client.get_products") as mock_products:
            mock_products.return_value = ["Product-1", "Product-2", "Product-3]
            yield


    @pytest.fixture
    def another_fixture(mock_api_call):
        ...

De nuevo, esto presenta un problema para linters como Pylint, que emitirá un
error como el siguiente:

`W0621: Redefining name 'mock_api_call' from outer scope (line 9) (redefined-outer-name)`

Esto es porque estamos usando el mismo nombre tanto en la función de fixture
como en los argumentos. Podemos solucionarlo y eliminar el error (correcto, por
otra parte) del linter asignando el nombre en el decorador de `fixture`:

    :::python
    @pytest.fixture(name="mock_api_call")
    def fixture_mock_api_call():
        with patch("project.api_client.get_products") as mock_products:
            mock_products.return_value = ["Product-1", "Product-2", "Product-3]
            yield


    @pytest.fixture
    def another_fixture(mock_api_call):
        ...

De esta forma, el error (y el código erróneo) desaparece.

## Sustituir variables de entorno en los tests

En caso de que nuestro proyecto use variables de entorno para su configuración,
podemos sobreescribirlas en nuestros tests de Pylint de las siguientes maneras:

### Fixture con variables de entorno para tests

En `conftest.py`:

    :::python
    import os
    from unittest.mock import patch

    import pytest


    @pytest.fixture(name="environment_dict")
    def fixture_environment_dict():
        yield {
            "API_URL": "https://example.com/api/v1/",
            "API_TOKEN": "oSTgtq4v2bERIXIplaeI6YNvkkBOXvoQT",
        }


    @pytest.fixture
    def mock_settings_env(environment_dict):
        with patch.dict(os.environ, environment_dict):
            yield

En `test_api.py`:

    :::python
    @pytest.mark.usefixtures("mock_settings_env")
    def test_api_url_invalid():
        with pytest.raises(SomeError) as error:
            # Do something that will cause an exception...

        error_message = str(error.value)
        assert "Some error message from the exception" in error_message

### Sustitución de las variables de entorno en el propio test

En `test_api.py`:

    :::python
    @patch.dict(os.environ, {"API_TOKEN": "oSTgtq4v2bERIXIplaeI6YNvkkBOXvoQT"}, clear=True)
    def test_api_url_mandatory():
        with pytest.raises(SomeError) as error:
            # Do something that will cause an exception...

        error_message = str(error.value)
        assert "Some error message from the exception" in error_message

El parámetro `clear=True` del decorador `@patch.dict` limpia todas las variables
de entorno y deja sólo el diccionario proporcionado.
