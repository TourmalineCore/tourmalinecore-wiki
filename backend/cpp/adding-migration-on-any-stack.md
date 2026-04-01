# Migration with Alembic

If your stack with framework doesent work with database migrations, you can use Alembic as migration layer to Database.

## Installing

Here **python** enviroment is requriment for properly working of Alembic.
So you need to instal **pip**:

<details>
<summary>Variant for Ubuntu systems</summary>
`apt-get update`
and
`apt-get install pip`
</details>

<details>
<summary>Variant for Windows systems</summary>
Need to instal **python** first by the installer from [official site](https://www.python.org/downloads/).
After installation puthon and if pip automaticly doesent installed, you need to download get-pip script and rin it.
Here a [get-pip](https://bootstrap.pypa.io/get-pip.py) script, after downloading just run `py get-pip.py`.

In any issueses case please read https://github.com/pypa/pip/issues for your problem
</details>

Make sure that **pip** is installed and install alembic packages with `pip install alembic psycopg2-binary`.

## Configuration

For properly working of Alembic, you need to create enviroment. Run these script in folder where you want to store all files with migrations `alembic init migrations`. In folder must created next tree:
```bash
.
├── alembic.ini
└── migrations
    ├── env.py
    ├── README
    ├── script.py.mako
    └── versions
```

In file `alembic.ini` on 89 line you must see `sqlalchemy.url = driver://user:pass@localhost/dbname` it is need to replace with your connection string to database, for example `sqlalchemy.url = pgsql://postgress:postgress@localhost/dbname`.

In file `env.py` on 21 line need to replace `target_metadata = None` to:
```python
import sys
import os
sys.path.append(os.path.abspath(os.path.join(os.path.dirname(__file__), '..', 'models')))
from to_do import Base
target_metadata = Base.metadata
```
To automaticly collecting models from `models` folder.
> Make sure you are created `models` folder in alembic enviroment folder.

> <details>
> <summary>Reading connection from .env file</summary>
> You may delete line with sqlalchemy.url from `alembic.ini` and define connection url direct in `env.py` by the next code snippet.
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

In created earlier folder `models` need to create all models from your orm. For model describing alembic uses **sqlalchemy**. Its automaticly installing along with alembic.

For example in article will be used `to-do` model from [integrating odb orm](integrating-odb-orm.md#describing-models) article.

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

More about describing models you may read in official [documentation](https://docs.sqlalchemy.org/en/20/tutorial/metadata.html#setting-up-metadata-with-table-objects).

## Creating and updating migration

For creating or updating migrations you must have active and [coonfigured](#configuration) database.
If connection is established u need to run command for create migration `alembic revision --autogenerate -m '<name of migration>'` and for apply new migrations run the `alembic upgrade head`.

## References

* [Python](https://www.python.org/)
* [PIP Documentation](https://pip.pypa.io/en/stable/installation/)
* [Sqlalchemy Documentation](https://docs.sqlalchemy.org/en/20/index.html)
* [Alembic Documentation](https://alembic.sqlalchemy.org/en/latest/)