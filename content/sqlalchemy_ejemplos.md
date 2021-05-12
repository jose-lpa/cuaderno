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
