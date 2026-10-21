# Migrations with Alembic

If your framework or stack does not support database migrations, you can use Alembic as a migration layer for your database.

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Installing](#installing)
- [Configuration](#configuration)
- [Describing Models](#describing-models)
- [Creating and Updating Migrations](#creating-and-updating-migrations)
- [References](#references)

<!-- TOC end -->

## Installing

A **python** environment is required for Alembic to work properly.  
So, you need to install **pip**:

<details>
<summary>Variant for Ubuntu systems</summary>
`apt-get update`
and
`apt-get install pip`
</details>

<details>
<summary>Variant for Windows systems</summary>
You need to install **python** first using the installer from the [official site](https://www.python.org/downloads/).
After installation, if python and pip were not installed automatically, you need to download the get-pip script and run it.
Here is the [get-pip](https://bootstrap.pypa.io/get-pip.py) script; after downloading it, just run `py get-pip.py`.

If you run into any issues, please check [https://github.com/pypa/pip/issues](https://github.com/pypa/pip/issues) for your problem.
</details>

Make sure that **pip** is installed, then install the Alembic packages with `pip install alembic psycopg2-binary`.

## Configuration

For Alembic to work properly, you need to create an environment. Run this script in the folder where you want to store all migration files: `alembic init migrations`. The following tree should be created in that folder:

```bash
.
├── alembic.ini
└── migrations
    ├── env.py
    ├── README
    ├── script.py.mako
    └── versions
```

In the `alembic.ini` file, on line 89, you should see `sqlalchemy.url = driver://user:pass@localhost/dbname`. You need to replace it with your database connection string, for example: `sqlalchemy.url = pgsql://postgress:postgress@localhost/dbname`.

In the `env.py` file, on line 21, you need to replace `target_metadata = None` with:

```python
import sys
import os
sys.path.append(os.path.abspath(os.path.join(os.path.dirname(__file__), '..', 'models')))
from to_do import Base
target_metadata = Base.metadata
```

This allows Alembic to automatically collect models from the `models` folder.

> Make sure you have created a `models` folder in the Alembic environment folder.

> <details>
> <summary>Reading connection from .env file</summary>
> You may delete the line with sqlalchemy.url from `alembic.ini` and define the connection URL directly in `env.py` using the following code snippet.
>
> ```python
>  def get_database_url():
>     """Getting the Database URL from Environment Variables"""
>     host = os.getenv("POSTGRES_HOST")
>     port = os.getenv("POSTGRES_PORT")
>     user = os.getenv("POSTGRES_USER")
>     password = os.getenv("POSTGRES_PASSWORD")
>     dbname = os.getenv("POSTGRES_DB")
>     
>     return f"postgresql://{user}:{password}@{host}:{port}/{dbname}"
> 
> url = get_database_url()
> ```
> And deleting `url = config.get_main_option("sqlalchemy.url")`.
> </details>

## Describing Models

In the previously created `models` folder, you need to create all models from your ORM. For describing models, Alembic uses **sqlalchemy**. It is installed automatically together with Alembic.

For example, this article uses the `to-do` model from the [integrating odb orm](integrating-odb-orm.md#describing-models) article.

```python
from sqlalchemy import create_engine, Column, BigInteger, Integer, String, ForeignKey, DateTime
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class ToDo(Base):
    __tablename__ = "todo"

    id = Column(BigInteger, primary_key=True)
    name = Column(String(255), nullable=False)
    createdAtUtc = Column(BigInteger, nullable=False)
    deletedAtUtc = Column(BigInteger, nullable=True)
```

You can read more about describing models in the official [documentation](https://docs.sqlalchemy.org/en/20/tutorial/metadata.html#setting-up-metadata-with-table-objects).

## Creating and Updating Migrations

To create or update migrations, you must have an active and [configured](#configuration) database.
If the connection is established, run the following command to create a migration: `alembic revision --autogenerate -m '<name of migration>'`.
To apply new migrations, run `alembic upgrade head`.

## References

* [Python](https://www.python.org/)
* [PIP Documentation](https://pip.pypa.io/en/stable/installation/)
* [Sqlalchemy Documentation](https://docs.sqlalchemy.org/en/20/index.html)
* [Alembic Documentation](https://alembic.sqlalchemy.org/en/latest/)