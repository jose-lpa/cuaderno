Title: Ejemplos y trucos de consultas a BD con SQLAlchemy
Date: 2021-05-12 16:32
Category: SQL
Tags: python,sqlalchemy,consultas,ejemplos
Slug: ejemplos-sqlalchemy
Authors: José L. Patiño-Andrés
Summary: Ejemplos de consultas y trucos con el ORM de SQLAlchemy

## Uso de `DISTINCT` junto con `ORDER BY` para diferentes columnas

En ocasiones es necesario hacer consultas que nos devuelvan un único resultado
para el valor de una determinada columna, y además ordenar dichos resultados en
base a una columna totalmente diferente.

Este escenario presenta un problema: las columnas que hayan en la cláusula 
`ORDER BY` deben obligatoriamente ser las mismas que las de la cláusula 
`DISTINCT`. Y probablemente no queramos, o no podamos, poner las mismas columnas
en ambas cláusulas, con lo cual nuestra consulta devolverá un error de este
tipo:

    :::python
    sqlalchemy.exc.ProgrammingError: (psycopg2.errors.InvalidColumnReference) SELECT DISTINCT ON expressions must match initial ORDER BY expressions

Veamos un ejemplo:

    :::python
    class Warehouse(Base):
        __tablename__ = "warehouse"

        id = Column(
            UUID(as_uuid=True), 
            server_default=func.uuid_generate_v4(), 
            primary_key=True
        )
        name = Column(String, index=True, nullable=False)
        company_id = Column(Integer, nullable=False)
        created_at = Column(
            DateTime(timezone=True), 
            server_default=func.now()
        )
        updated_at = Column(
            DateTime(timezone=True), 
            server_default=func.now()
        )

        item = relationship("Item", back_populates="warehouse")


    class Item(Base):
        __tablename__ = "item"

        warehouse_id = Column(
            UUID(as_uuid=True), 
            ForeignKey("survey.id"), 
            index=True, 
            nullable=False, 
            primary_key=True
        )
        stock = Column(Integer, nullable=False)
        name = Column(String, nullable=False)
        category = Column(String, nullable=False)

        warehouse = relationship("Warehouse", back_populates="item")


Con estos modelos, si queremos hacer una consulta que nos devuelva **los 
`Warehouse` que tengan `Item`'s de nombre `name = "Smartphone"`**, mostrando
**únicamente un solo `Warehouse` de cada compañía, `distinct(Warehouse.company_id)`**
que será exactamente **el `Warehouse` de esa compañía que se creó primero, `created at = my_date`**,
haríamos algo así:

    :::python
    from sqlalchemy import desc, orm


    def get_warehouses(session: orm.Session):
        q = session.query(Warehouse).join(Item.warehouse).filter(
            Item.name = "Smartphone"
        ).distinct(Warehouse.company_id)

        return q.order_by(desc(Warehouse.created_at))

Lo cual nos devolvería el error antes señalado.

Para sortear esta situación, tenemos 2 opciones. Ambas hacen lo mismo: envolver
la consulta en un `SELECT` y, a partir de sus resultados, hacer el `ORDER BY`:

### Uso del método `Query.from_self()` (SQLAlchemy 1.4, OBSOLETO)

El método [`Query.from_self()`](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.from_self)
es una solución rápida al problema.

Este método se debe usar únicamente en versiones de SQLAlchemy 1.4 o inferiores,
ya que ha sido declarado obsoleto y propuesto para ser eliminado.

    :::python
    def get_warehouses(session: orm.Session):
        q = session.query(Warehouse).join(Item.warehouse).filter(
            Item.name = "Smartphone"
        ).distinct(Warehouse.company_id)

        return q.from_self().order_by(desc(Warehouse.created_at))


### Uso de la función `aliased()` (SQLAlchemy 1.4+, RECOMENDADO)

La función de ORM [`aliased()`](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased)
es un poco [más elaborada](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased):

    :::python
    from sqlalchemy import desc, orm
    from sqlalchemy.orm import aliased


    def get_warehouses(session: orm.Session):
        subq = session.query(Warehouse).join(Item.warehouse).filter(
            Item.name = "Smartphone"
        ).distinct(Warehouse.company_id).subquery()

        qa = aliased(Warehouse, subq)

        return session.query(qa).order_by(desc(qa.created_at))


## Asignación automática de UUIDs en PostgreSQL

En PostgreSQL, podemos hacer que la base de datos genere automáticamente un UUID
para una columna `uuid` de una nueva fila sin necesidad de especificarla de
forma manual, igual a como se autogeneran los tipos [`serial` o `bigserial`](https://www.postgresql.org/docs/14/datatype-numeric.html#DATATYPE-SERIAL)
comúnmente usados para identificadores.

Para ello, primero debemos instalar la extensión [uuid-ossp](https://www.postgresql.org/docs/14/uuid-ossp.html)
en nuestra base de datos:

    :::postgresql
    CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

Y ahora en SQLAlchemy ya podemos usar por ejemplo la función `uuid_generate_v4`:

    :::python
    from sqlalchemy import Column, String
    from sqlalchemy.dialects.postgresql import UUID
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.sql import func

    
    Base = declarative_base()


    class Items(Base):
        __tablename__ = "items"

        uid = Column(UUID(as_uuid=True), server_default=func.uuid_generate_v4(), primary_key=True)
        name = Column(String(50), unique=True, nullable=False)


## Ejemplos de relación "Many to Many"

### Tabla asociativa

El primer ejemplo es el más sencillo: se trata de una tabla "many to many" que 
relaciona de manera directa y simple a otras dos tablas. Las operaciones de
creación, borrado, etcétera se hacen a través de las referencias a esta tabla.

    :::python
    from sqlalchemy import Column, ForeignKey, Integer, Table, String
    from sqlalchemy.dialects.postgresql import ARRAY, UUID
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import relationship
    from sqlalchemy.sql import func

    
    Base = declarative_base()


    organization_users = Table(
        "organization_users",
        Base.metadata,
        Column("organization_uid", ForeignKey("organization.uid", ondelete="CASCADE"), primary_key=True),
        Column("user_uid", ForeignKey("user.uid", ondelete="CASCADE"), primary_key=True)
    )


    class Organization(Base):
        __tablename__ = "organization"

        uid = Column(UUID(as_uuid=True), server_default=func.uuid_generate_v4(), primary_key=True)
        name = Column(String(50), unique=True, nullable=False)

        users = relationship(
            "User", secondary=organization_users, back_populates="organizations", cascade="all, delete"
        )


    class User(Base):
        __tablename__ = "app_user"

        uid = Column(UUID(as_uuid=True), server_default=func.uuid_generate_v4(), primary_key=True)
        name = Column(String(50), nullable=False)
        email = Column(String(150), nullable=False, unique=True)

        organizations = relationship(
            "Organization", secondary=organization_users, back_populates="users", passive_deletes=True
        )

Con este esquema de base de datos, podemos hacer consultas como las siguientes:

    :::python
    # Crear Organization
    organization = Organization(name="VASS Logic Ltd.")
    db_session.add(organization)
    db_session.commit()

    # Crear User
    user = User(name="José L. Patiño", email="jose@sharklasers.com")
    db_session.add(user)
    db_session.commit()

    # Asociar User con Organization
    user.organizations.append(organization)
    db_session.commit()

    # Si borras un User, por supuesto la Organization permanecerá
    db_session.delete(user)
    db_session.commit()

    assert db_session.scalar(select(func.count()).select_from(Organization)) == 1

    # Pero si borras una Organization, todos sus Users TAMBIÉN SON BORRADOS
    db_session.delete(organization)
    db_session.commit()

    assert db_session.scalar(select(func.count()).select_from(User)) == 0

### Objeto asociativo

El Objeto Asociativo es una estrategia similar a la de Tabla Asociativa, pero
nos da la ventaja de que podemos **añadir campos extra a esa tabla intermedia**.

Esto nos podría ser muy útil en ese sentido, si necesitásemos tener una
relación "many to many" pero en la tabla intermedia tener más datos aparte de
los simples Foreign Keys hacia los objetos relacionados.

Se ve mucho mejor con un ejemplo. Si necesitásemos un sistema de "roles de
usuario", donde un usuario puede pertenecer a varias organizaciones y en cada
una de ellas se le asigna un rol de usuario, podríamos hacer esto:

    :::python
    from sqlalchemy import Column, ForeignKey, Integer, String
    from sqlalchemy.dialects.postgresql import ARRAY, UUID
    from sqlalchemy.orm import relationship
    from sqlalchemy.sql import func

    from user_management.core.database import Base


    class OrganizationUser(Base):
        __tablename__ = "organization_user"
        organization_uid = Column(ForeignKey("organization.uid", ondelete="CASCADE"), primary_key=True)
        user_uid = Column(ForeignKey("user.uid", ondelete="CASCADE"), primary_key=True)
        role = Column(String(15))

        user = relationship("User", back_populates="organizations", cascade="all, delete")
        organization = relationship("Organization", back_populates="users")


    class Organization(Base):
        __tablename__ = "organization"

        uid = Column(UUID(as_uuid=True), server_default=func.uuid_generate_v4(), primary_key=True)
        name = Column(String(50), unique=True, nullable=False)

        users = relationship("OrganizationUser", back_populates="organization", cascade="all, delete")


    class User(Base):
        __tablename__ = "user"

        uid = Column(UUID(as_uuid=True), server_default=func.uuid_generate_v4(), primary_key=True)
        name = Column(String(50), nullable=False)
        email = Column(String(150), nullable=False, unique=True)

        organizations = relationship("OrganizationUser", back_populates="user", cascade="all, delete")

Esto funcionaría igual al ejemplo anterior, pero ahora deberíamos crear la
relación usando el objeto `OrganizationUser`.

    :::python
    # Crear Organization
    organization = Organization(name="VASS Logic Ltd.")
    db_session.add(organization)
    db_session.commit()

    # Crear User
    user = User(name="José L. Patiño", email="jose@sharklasers.com")
    db_session.add(user)
    db_session.commit()

    # Asociar User con Organization
    organization_user = OrganizationUser(organization=organization, user=user)
    user.organizations.append(organization_user)
    db_session.commit()

    # El resto de características son similares a las del primer ejemplo
