#  Flask-SQLAlchemy Setup and CRUD Technical Lesson

## The Scenario

You have just joined a small tech startup called Pawfect Match, which specializes in helping shelters and foster homes connect pets with new families. The team is building an internal tool to keep track of pets currently available for adoption.

The current setup is messy — volunteers are trying to manage everything in spreadsheets, which often leads to errors and duplicate records. Your first task as a new junior developer is to help them set up a proper web-based backend.

You’ll build a basic Flask application using Flask-SQLAlchemy to define a Pet model, set up the database, and implement basic CRUD operations. This foundational system will allow the team to easily add, view, update, and remove pets from the adoption list — all without having to write raw SQL queries!

---

## Tools & Resources

- [SQLAlchemy](https://www.sqlalchemy.org/)
- [Alembic](https://alembic.sqlalchemy.org/en/latest/)
- [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/3.0.x/)
- [Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/)
- [SQLite Viewer VSCode Extension](https://marketplace.visualstudio.com/items?itemName=qwtel.sqlite-viewer)
- [Metadata - SQLAlchemy 2.0 Documentation](https://docs.sqlalchemy.org/en/20/core/metadata.html)

---

## Instructions

### Set Up

This lesson is a code-along, so fork and clone the repo.

`Pipfile` has some new dependencies that we'll use in this
lesson:`flask-sqlalchemy`and`flask-migrate`. Run `pipenv install`to install the
dependencies and `pipenv shell` to enter your virtual environment before running
your code.

```console
$ pipenv install
$ pipenv shell
```

Let's use the `tree` command to view the directory structure. MacOS users can
install this with the command `brew install tree`. WSL and Linux users can run
`sudo apt-get install tree` to download it.

```console
$ tree
```

The `server` folder initially contains two files, `app.py` and `models.py`

```text
├── CONTRIBUTING.md
├── LICENSE.md
├── Pipfile
├── Pipfile.lock
├── README.md
└── server
    ├── app.py
    └── models.py
```

- `app.py` contains the code to configure and connect a web server to a
  database.
- `models.py` contains a **model** named `Pet`.

### Step 1: Define Model

A **model class**, also referred to simply as a **model**, is a Python class
that (1) defines a new Python data type, and (2) defines database metadata to
describe an SQL table. We can treat a model class like any other Python class,
meaning we can create instances, invoke methods, etc. Additionally,
Flask-SQLAlchemy performs object-relational mapping between the model class and
an associated database table, while Flask-Migrate uses changes to the model
class to evolve a database schema.

Defining a model with Flask-SQLAlchemy requires a Python class with four key
traits:

- Inheritance from the `db.Model` class.
- A `__tablename__` class attribute.
- One or more class attributes assigned to the type `db.Column`.
- One class attribute specified to be the table's primary key.

Let's take a look at a model defined by the Python class `Pet` in
`server/models.py`:

```py
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import MetaData

# contains definitions of tables and associated schema constructs
# read more about Metadata using the link at the bottom of the page
metadata = MetaData()

# create the Flask SQLAlchemy extension
db = SQLAlchemy(metadata=metadata)

# define a model class by inheriting from db.Model.
class Pet(db.Model):
    __tablename__ = 'pets'

    id = db.Column(db.Integer)
    name = db.Column(db.String)
    species = db.Column(db.String)
```

## Step 2: Add Column Constraints

Next, add column constraints to our model specify primary_key and any other constraints we want such as string character limits.

We'll set id as the primary key and not allow names to be more than 50 characters.

```py
class Pet(db.Model):
    __tablename__ = 'pets'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))
    species = db.Column(db.String)
```

- The `Pet` class is declared as a subclass of `db.Model`.
- The database table is named `pets`.
- The database table has 3 columns:
  - the `id` column is the primary key
  - the `name` column stores a string
  - the `species` column stores a string

### Step 2: Configure the Flask-SQLAlchemy extension

The file `app.py`:

- Creates a Flask application object named `app`
- Configures the database connection to a local file `app.db`
- Creates an object named `migrate` to manage schema migrations (versions)
- Initializes the SQLAlchemy extension with the application

```py
# server/app.py

from flask import Flask
from flask_migrate import Migrate

from models import db

# create a Flask application object
app = Flask(__name__)

# configure a database connection to the local file app.db
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

# disable modification tracking to use less memory
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# create a Migrate object to manage schema modifications
migrate = Migrate(app, db)

# initialize the Flask application to use the database
db.init_app(app)


if __name__ == '__main__':
    app.run(port=5555, debug=True)

```

Eventually we will add routes to `app.py` to implement CRUD operations on the
`pets` table. First, we need to step through the process of creating the
database file `app.db` and adding the `pets` table to the database.

Let's configure the `FLASK_APP` and `FLASK_RUN_PORT` environment variables
before proceeding with the database migration:

```console
$ export FLASK_APP=app.py
$ export FLASK_RUN_PORT=5555
```

### Step 4: Define schema and generate migrations

We know how to write SQL statements to define and modify a database schema. For
example, we used the SQL `create table` statement to define a database table,
and the `update table` statement to modify the structure of an existing table.

A migration is a set of SQL statements that tells a database how to move from an
old schema to a new one (schema migration), and also how to move a database
entirely from one server to the next (server migration). We will only be
focusing on the former, **schema migrations**.

A schema migration is also performed when we first define a schema, i.e. to
create the initial tables and other database structures. Thankfully, we can use
the Flask extension **Flask-Migrate** to automatically define the schema based
on models contained in `server/models.py`!

Change into the `server` directory:

```console
$ cd server
```

Type the following command to create a migration environment:

```console
$ flask db init
```

The `server` directory should now contain two new directories `instance` and
`migrations`:

```text
..
├── CONTRIBUTING.md
├── LICENSE.md
├── Pipfile
├── Pipfile.lock
├── README.md
├── notes
└── server
    ├── app.py
    ├── instance
    ├── migrations
    │   ├── README
    │   ├── alembic.ini
    │   ├── env.py
    │   ├── script.py.mako
    │   └── versions
    └── models.py
```

The `instance` folder will eventually hold the database file `app.db`.

The `migrations` folder contains a migration environment:

- `alembic.ini` defines environment configuration options.
- `env.py` defines and instantiates a SQLAlchemy engine, connects to that
  engine, starts a transaction, and calls the migration engine.
- `script.py.mako` is a template that is used when creating a migration- it
  defines the basic structure of a migration.
- `versions` is a directory to hold migration scripts.

Next we will use the command `flask db migrate -m message` to generate a
migration script in the `migrations/versions` directory. The `-m message` is an
optional flag that lets us add a message describing the migration.

Type the following command to generate an initial migration:

```console
$ flask db migrate -m "Initial migration."
```

. The command results in two new files:

- The database `app.db` is added to the `instance` directory.
- A Python migration script of the form `###_message.py` is added to the
  `migrations/versions` directory.
  - `###` is a random version number.
  - `message` is the text specified with the `-m` flag.

```text
├── CONTRIBUTING.md
├── LICENSE.md
├── Pipfile
├── Pipfile.lock
├── README.md
├── notes
└── server
    ├── app.py
    ├── instance
    │   └── app.db
    ├── migrations
    │   ├── README
    │   ├── alembic.ini
    │   ├── env.py
    │   ├── script.py.mako
    │   └── versions
    │       └── ###_message.py
    └── models.py
```

Open the migration script in the editor. You'll see it contains functions
`upgrade()` and `downgrade()` that create and drop the `pets` table. The
`upgrade()` function is generated using the schema details defined by the `Pet`
model class.

## Step 5: Upgrade the database

Finally, type the following to run the `upgrade()` function and create the
`pets` table:

```console
$ flask db upgrade head
```

The `head` is optional and refers to the most recent migration version.

Open the database file `app.db` using a VSCode extension for viewing SQLite
database. The image below shows the database using the `SQLite Viewer`
extension. Assuming you've installed the extension, right-click on `app.db`,
then select `Open With.../SQLite Viewer`. Confirm the database contains a new
table named `pets` with columns as defined by the `Pet` model class.

![new pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/pet_table.png)

## Step 6: Enter the Flask Shell

Let's see how to persist data about a pet. Recall from the previous lessons
about ORM that we don't actually save a Python object to the database. Instead,
we save the object's attributes as a new row in a table.

We can interact with our code in the Python shell or an `ipdb` session, but
working with a web framework presents a bit of a conundrum: the application
isn't running! Thankfully, Flask comes equipped with an interactive shell that
runs a development version of an application. Inside this shell, we can interact
with models, views, contexts, and the database.

If you're not there already, navigate to the `server` directory, then enter the
command `flask shell`:

```console
$ flask shell
>>>
```

You will type commands after the `>>>` prompt.

First, let's import the necessary `db` database object and the `Pet` model:

```console
>>> from models import db, Pet
```

## Step 7: Use Flask Shell to add data with `add()` and `commit()`

Let's add a row to the `pets` table for a dog named "Fido". The steps to add a
row are as follows:

1. Create a new instance of the model class `Pet`.
2. Add the `Pet` instance to the current database session.
3. Commit the transaction and apply the changes to the database.

The first step is creating the `Pet` instance. Type the following Python
assignment statement in the Flask shell:

```console
>>> pet1 = Pet(name = "Fido", species = "Dog")
```

An instance of `Pet` is created, however, the object has not been persisted to
the database.

Let's confirm the `name` and `species` attributes have been assigned values, but
`id` does not yet have a value:

```console
>>> pet1.name
'Fido'
>>> pet1.species
'Dog'
>>> pet1.id
>>>
```

The string representation returned by the implicit call to the `__repr__`
function shows the id as `None`, confirming no value has been assigned:

```console
>>> pet1
<Pet None, Fido, Dog>
```

NOTE: The `id` won't be assigned until the `Pet` instance has been added to the
database.

Persisting an object to the database requires a database session, which is an
object that manages database transactions. A transaction is a sequence of SQL
statements that are processed as an atomic unit. This means that either all SQL
statements in the transaction are either applied (committed) or they are all
undone (rolled back) together.

This is especially important if statements that occur in a sequence depend on
previous statements executing properly. The workflow for a transaction is
illustrated in the image below:

![Workflow for a successful transaction. Shows that after a transaction begins,
the state of the database is recorded, then statements are executed, then the
transaction is committed if all statements are successful.](https://curriculum-content.s3.amazonaws.com/python/esal_0401.png "successful transaction")

If any of the SQL statements in a transaction fail to execute properly, the
database will be rolled back to the state recorded at the beginning of the
transaction and the process will end, returning an error message. A committed
transaction ensures all statements were executed in sequence and to completion.

Flask-SQLAlchemy provides the `db.session` object through which we can manage
changes to the database such as table row insertions, updates, and deletions.

Let's add the pet object to the database session using the `db.session.add()`
method. Type the following in the Flask shell:

```console
>>> db.session.add(pet1)
```

This method call will issue an SQL INSERT statement, but the `id` attribute of
the `Pet` instance in the Python application is still undefined because we have
not yet committed the current transaction. We need to call the
`db.session.commit()` method to commit the transaction and ensure the new row
was inserted in the database table.

```console
>>> db.session.commit()
```

Check the `pets` table to confirm a new row was added. If you are using SQLite
Viewer, you may need to press the refresh button to see the new row:

![first row inserted in pet](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/insert_dog1.png)

When the transaction is committed and the row is inserted in the `pets` table,
the `id` of the local `Pet` instance is assigned the primary key value from the
new row:

```console
>>> pet1.id
1
>>>
```

Let's add another pet to the database. Type each Python statement one at a time
at the Flask shell prompt:

```console
>>> pet2 = Pet(name = "Whiskers", species = "Cat")
>>> db.session.add(pet2)
>>> db.session.commit()
```

Refresh the view in the SQLite Viewer to confirm a new row was inserted in the
`pets` table for the cat named "Whiskers":

![second row inserted in pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/insert_cat.png)

In the Flask shell, we can confirm the `id` attribute is assigned for the newly
persisted object:

```console
>>> pet2.name
'Whiskers'
>>> pet2.species
'Cat'
>>> pet2.id
2
>>> pet2
<Pet 2, Whiskers, Cat>
```

## Step 8: Use Flask shell to query() data

### query()

We can query all the rows in the table associated with the `Pet` model as shown:

```console
>>> Pet.query.all()
[<Pet 1, Fido, Dog>, <Pet 2, Whiskers, Cat>]
```

How did the `Pet` class get a `query` attribute? `Pet` inherits it from
`db.Model`! The `all()` function says to return every row from the query result.

If we just want just the first row returned from a query, use the `first()`
function:

```console
>>> Pet.query.first()
<Pet 1, Fido, Dog>
```

### filter()

We can filter rows using the `filter` function. The function takes a boolean
expression as an argument that is evaluated against each model instance returned
from the query. For example, if we want to filter each pet by species:

```console
>>> Pet.query.filter(Pet.species == 'Cat').all()
[<Pet 2, Whiskers, Cat>]
```

If we want pets whose name starts with the letter 'F':

```console
>>> Pet.query.filter(Pet.name.startswith('F')).all()
[<Pet 1, Fido, Dog>]
```

### filter_by()

The `filter` function is powerful in that you can pass any boolean expression to
test on a model instance. However, we often want to just look for rows having a
particular value in a column. The `filter_by` function is useful for that. The
criteria passed as a function argument takes a single equal sign. For example,
to get all cats:

```console
>>> Pet.query.filter_by(species = 'Cat').all()
[<Pet 2, Whiskers, Cat>]
```

We can filter by the primary key `id` to get a specific row:

```console
>>> Pet.query.filter_by(id = 1).first()
<Pet 1, Fido, Dog>
```

### get()

If you want to access a certain row by its primary key, use
`db.session.get(Model, id)`. It will return the row with the given primary key,
or `None` if it doesn't exist. The main advantage is caching: SQLAlchemy's
session maintains an identity map, so if the object with the specified ID is
already in the session, it will return that instance without hitting the
database again.

```console
>>> pet = db.session.get(Pet,1)
>>> pet
<Pet 1, Fido, Dog>
>>> pet is None
False
>>> pet = db.session.get(Pet,20)
>>> pet
>>> pet is None
True
>>>
```

### order_by()

By default, results from any database query are ordered by their primary key.
The `order_by()` method allows us to sort by any column. To sort in ascending
order of species:

```console
>>> Pet.query.order_by('species').all()
[<Pet 2, Whiskers, Cat>, <Pet 1, Fido, Dog>]
```

### `func`

Importing `func` from `sqlalchemy` gives us access to common SQL operations
through functions like `sum()` and `count()`.

```console
>>>from sqlalchemy import func
```

As these operations act upon columns, we carry them out through wrapping a
`Column` object passed to the `query()` method. Note we are invoking the `query`
function on the session, rather than accessing the `query` attribute inherited
from `db.Model`:

```console
>>> db.session.query(func.count(Pet.id)).first()
(2,)
```

It is best practice to call these functions as `func.operation()` rather than
their name alone because many of these functions have name conflicts with
functions in the Python standard library, such as `sum()`.

## Step 9: Use Flask Shell to update data

When we assign a new attribute value to a Python object that has been persisted
to the database, the associated table row **does not** automatically get
updated.

We need to perform the following steps to update a row in the `pets` table:

1. Update one or more attribute values of a `Pet` instance.
2. Commit the transaction to apply the changes to the database.

```console
>>> pet1
<Pet 1, Fido, Dog>
>>> pet1.name = "Fido the mighty"   # this does not update the table row
>>> pet1
<Pet 1, Fido the mighty, Dog>
>>> db.session.commit()             # commit the UPDATE statement
```

We can see the table row is updated once the transaction is committed:

![update row in pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/update_pet.png)

## Step 10: Use Flask Shell to delete() data

The `db.session.delete()` function is used to delete the row associated with an
object:

```console
>>> db.session.delete(pet1)
>>> db.session.commit()
```

Query the `Pet` model to confirm the row was deleted:

```console
>>> Pet.query.all()
[<Pet 2, Whiskers, Cat>]
```

We can also check the table using the SQLite Viewer:

![delete row from pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/delete_pet.png)

If you want to delete all table rows, call the function `Pet.query.delete()`.
The function returns the number of rows deleted. Make sure you commit the
transaction to persist the deleted row.

```console
>>> Pet.query.delete()
1
>>> db.session.commit()
```

We can use the Flask shell to confirm there are no pets in the table:

```console
>>> Pet.query.all()
[]
```

The SQLite Viewer also shows the empty table:

![new pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/pet_table.png)

## Step 11: Exit the Flask shell

You can exit the Flask shell and return to the command line prompt using
`CTRL + D` or the `exit()` function:

```console
>>> exit()

$
```

## Step 12: Commit, Push, and Merge to GitHub

* Commit and push your code

```bash
git add .
git commit -m "create pets table with some started data"
git push
```

* If you created a separate feature branch, remember to open a PR on main and merge.
* Submit your repo with the updated code to Canvas.

### Task 4: Document and Maintain

Best Practice documentation steps:
* Add comments to the code to explain purpose and logic, clarifying intent and functionality of your code to other developers.
* Update README text to reflect the functionality of the application following https://makeareadme.com. 
  * Add screenshot of completed work included in Markdown in README.
* Delete any stale branches on GitHub
* Remove unnecessary/commented out code
* If needed, update git ignore to remove sensitive data

---


## Considerations

### Column Constraints

The `Pet` model maps to a basic database table with 3 columns (`id`, `name`,
`species`). There are no column constraints other than the primary key `id`
and name cannot be more than 50 characters:

```py
# define a model class by inheriting from db.Model.
class Pet(db.Model):
    __tablename__ = 'pets'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))
    species = db.Column(db.String)
```

However, SQLAlchemy (and therefore Flask-SQLAlchemy) let's us define many types
of column constraints. For example, the `User` model below demonstrates some
common constraints:

```py
class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True,
                         nullable=False, index=True)
    email = db.Column(db.String(120), unique=True)
    verified = db.Column(db.Boolean, default=False)
```

- `id` is the primary key
- `username` is a unique string of length 80. Null values are not allowed. An
  index is set on the column to speed up queries when searched by this column.
- `email` is a unique string of length 120. Null values are allowed.
- `verified` is a boolean that defaults to `False` if a value is not given.

### Flask Shell

`flask shell` is a great tool for simple debugging and adding or updating a few
records. We want our app to handle many records though, which would take too
long to do by hand in the Flask shell. In subsequent lessons, we'll see how to
add routes to a Flask app to support full CRUD operations.