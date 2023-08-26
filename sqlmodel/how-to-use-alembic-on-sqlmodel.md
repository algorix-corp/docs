# How to use Alembic (Migration Tool) on SQLModel

* Notice that this document is based on [stackoverflow](https://stackoverflow.com/questions/68932099/how-to-get-alembic-to-recognise-sqlmodel-database-model).

1. Setup all (FastAPI+) SQLModel + Alembic Environment.
```bash
pip install fastapi sqlmodel alembic
```

2. Initialize Alembic.
> `migrations` is directory name where Alembic data is stored, the name can be changed but we ***strongly suggest*** you to use **migrations**.
```bash
alembic init migrations
```

3. Make SQLModel model
> By making `class Table(SQLModel, table=true)` class, you can add tables' information to SQLModel(SQLAlchemy) Metadata.
```python
from datetime import datetime
from typing import Optional

from sqlmodel import SQLModel, Field


class User(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    username: str
    password: str


class Item(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    description: Optional[str] = None
    owner_id: int = Field(foreign_key="user.id")
```

4. Import SQLModel on `./migrations/script.py.mako` in migrations directory.
```python
"""${message}

Revision ID: ${up_revision}
Revises: ${down_revision | comma,n}
Create Date: ${create_date}

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa
import sqlmodel  # << THIS LINE ADDED <<
${imports if imports else ""}

# revision identifiers, used by Alembic.
revision: str = ${repr(up_revision)}
down_revision: Union[str, None] = ${repr(down_revision)}

(...)
```

5. Load models and set target metadata on `./migrations/env.py`.
```python
from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

from models import *  # << THIS LINE ADDED <<

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

# Interpret the config file for Python logging.
# This line sets up loggers basically.
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
target_metadata = SQLModel.metadata  # << THIS LINE ADDED <<

(...)
```

6. Set Database Connection String in `./alembic.ini`.
```
(...)

# are written from script.py.mako
# output_encoding = utf-8

sqlalchemy.url = sqlite:///database.db  # << THIS LINE <<


[post_write_hooks]
# post_write_hooks defines scripts or Python functions that are run
# on newly generated revision scripts.  See the documentation for further

(...)
```

7. Run `alembic upgrade head` to follow-up current scheme.

8. If you want to revise the scheme, run `alembic revision --autogenerate -m "your message"`. It will make revision to `./migrations/versions/`.
> You still have to run **Step 7** to update the remote database's scheme.

---

Copyright 2023. Algorix LLC. All rights reserved.
