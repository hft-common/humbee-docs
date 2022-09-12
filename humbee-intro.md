
# Humbee Intro

Rules common to all packages:
=============================

*   Log at the top of a function something like “foo called”.
*   If a function has the `company_id` or `user_id` parameter, that must be logged
*   Results of all functions must be logged
*   Logs must be present after if and else statements
*   If a function has more than 5 parameters, use a DTO
*   If a function has more than 1 return value, use a DTO
*   Phone number must always be treated and stored as a string
*   NEVER push to main. Push to a branch called your_name-dev and then raise a PR. 
*   Always test your code in swagger first before pushing.
*   Read the README for any queries.
*   Follow the migration steps as written in the README.

Separate all code into 3 main folders:

1.  Logic
2.  Data
3.  API

Data folder
===========

Data folder has the plain sqlalchemy queries. Plain meaning it only has read queries, write queries and some joins.

General rules for the data folder:

*   Models must go in the model folder
*   Enums must go in the enum folder
*   Any and all queries must go into the data folder

*   Each model must have their own folder
*   Read queries and write queries must go into different folders

*   All queries must have the dbapi_exception_handler decorator
*   Read and write query packages should have the naming convention: modelname_read_queries and modelname_write_queries respectively

General rules for queries

*   All read queries must have the parameters `session=None` and `close_session=True`
*   All write queries must have the `session=None` and `commit=True` parameters
*   All query functions must get the DB session with the statement: `db = session if session else next(get_db())`
*   Log before committing and flushing.
*   If `commit=False`, the use `db.flush()`

The Logic folder
================

The logic folder will have your business logic code.

*   Each package will correspond to a module.
*   DTOs will go in the dtos folder
*   Joins can go in the logic folder

Standard structure for most logic functions should be:

``` python
# log function

default_log.debug(f“foo called with company_id={company_id}”)

# start session with db

db = next(get_db())

# Do processing

# If you are making calls to queries

result = some_dbapi_query(session=db)

default_log.debug(f“Result of some_dbapi_query: {result}”)

# close session

db.close()
```

Please remember that if a function has more than 5 parameters, use a DTO. If a function has more than one return value, use a DTO.

One example of using a return value DTO is this: Say you want to return different error messages based on different error conditions. You could have a FooRetvalDTO structured as:

```python
class FooRetvalDTO(BaseModel):
	is_error: bool
	error_message: str
	other_variables: Optional[None]
```

Here, `is_error` tells the caller function whether or not there is an error and then it can proceed further

The API Folder
==============

*   Functions in the API folder must have as LITTLE code as possible.
*   The must use the following decorators IN ORDER:

*   1) `@modulename_router.method(‘/foo’)`
*   2) `@frontend_api_generic_exception`
*   3) `@format_language`

*   All calls MUST have the first parameter as request: Request
*   All calls must have either the get_user_from_token dependency or similar, that locks the API call to authorized users only.
*   All API functions must return the response as the return value of standard_json_response
*   All messages at the API level must be constants defined in the localization folder
*   All request DTOs must raise a ValueError on error
*   Start of the function must be logged
*   HS and user APIs should be in separate files

DTO validations
===============

In general, see the `validators` package for existing validators for properties like company_type and phone_no.

If you believe that a validator is going to be used more than once, write a function in the validators package and in the DTO, use the return value of that function to do the validation

External services
=================

External services (third-party APIs) must go in the external services folder. They must have a class with a set of methods that make API calls on behalf of our code. DO NOT hard-code api routes in the logic folder or any other folder. All initialization code and api routes should go in the `__init__` function of that class and should be stored as variables of that class.

See the external_services package for examples.

Storing files
=============

*   All user files must be stored using S3
*   All inputs for files must be taken as base64 strings
*   The filename should be of the format “user_provided_filename” + md5sum of file.
*   Tables which store files must have the user provided filename as well as the actual filename with which we are storing
*   Use the get_aws_session() folder for an aws session

Secrets.json
============

*   No magic numbers. All constants should be in secrets.json, for example number of threads, s3_path, api keys, etc.
*   All of these should have a default value in `config.py` above the `if secrets_file.exists()` line.
*   The actual value should be read from secrets.json after this file

Using DBApiExceptionResponse
============================

Note that a DBApiExceptionResponse object evaluates to false. This can be used in several situations.

The DBApiExceptionseResponse will also store the error type as a string for further use.

Localization
============

All messages returned to the user must be defined as constants in the localization folder. The structure for localization files is as follows:

*   `api_package_name_api_strings.py` for English constants
*   `hindi/api_package_name_api_strings.py` for Hindi constants

Pagination
==========

In general, paginated routes will have the following parameters:

```python
foo(request: Request,
	f: list[str] = Query(...),
	o: list[str] = Query(...),
	page=1,
	page_size=10
)
```
Where the value of f will be passed to `filter_criteria_strings` and the value of o will be passed to `order_by_criteria_strings` of the `CustomGetQueryLogicDTO` with the proper model class.


