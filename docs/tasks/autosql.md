# `autosql` Task

## About

The `autosql` task lets you write a `SELECT` statement and SAYN then automates the data processing (i.e. table or view creation, incremental load, etc.) for you.

## Defining `autosql` Tasks In `models.yaml`

An `autosql` task is defined as follows:

!!! example "autosql task definition"
    ```yaml
    ...

    task_autosql:
      type: autosql
      file_name: task_autosql.sql
      materialisation: table
      destination:
        tmp_schema: analytics_staging
        schema: analytics_models
        table: task_autosql
    ...
    ```

An `autosql` task is defined by the following attributes:

* `type`: `autosql`.
* `file_name`: the name of the file **within the sql folder of the project's root**.
* `materialisation`: this should be either `table`, `view` or `incremental`. `table` will create a table, `view` will create a view. `incremental` will create a table and will load the data incrementally based on a delete key (see more detail on `incremental` below).
* `destination`: this sets the details of the data processing.
    * `tmp_schema`: specifies the schema which will be used to store any necessary temporary object created in the process. This is optional.
    * `schema`: is the destination schema where the object will be created. This is optional.
    * `table`: is the name of the object that will be created.
* `delete_key`: specifies the incremental process delete key. This is for `incremental` `materialisation` only.

## Using `autosql` In `incremental` Mode

If you do not want to have a full refresh of your tables, you can use the `autosql` task with `incremental` `materialisation`. This is extremely useful for large data volumes when full refresh would be too long.

SAYN `autosql` tasks with `incremental` materialisation require a `delete_key`. Please see below an example:

!!! example "autosql in incremental mode"
    ```yaml
    ...

    task_autosql_incremental:
      type: autosql
      file_name: task_autosql_incremental.sql
      materialisation: incremental
      destination:
        tmp_schema: analytics_staging
        schema: analytics_models
        table: task_autosql
      delete_key: dt
    ...
    ```

When using `incremental`, SAYN will do the following in the background:

1. Create a temporary table based on the incremental logic from the SAYN query.
2. Delete rows from the target table that are found in the temporary table based on the `delete_key`.
3. Load the temporary table in the destination table.

### Incremental SAYN parameters

There are three SAYN parameters which you can use for autosql tasks in incremental mode:

* `full_load`: controlled by the `-f` flag in the SAYN command. This will reload the whole table.
* `start_dt`: controlled by the `-s` flag in the SAYN command.
* `end_dt`: controlled by the `-e` flag in the SAYN command.

## Defining DDLs

You can define the following DDL parameters in the `autosql` task definition:

* indexes: the indexes to add on the table.
* primary_key: this should be added in the indexes section using the `primary_key` name for the index.
* columns: the list of columns as well as their type. If used, SAYN will enforce the types specified.
* permissions: the permissions you want to give to each role. You should map each role to the rights you want to grant separated by commas (e.g. SELECT, DELETE).

!!! example "autosql with DDL"
    ```yaml
    ...

    task_autosql:
      type: autosql
      file_name: task_autosql.sql
      materialisation: table
      destination:
        tmp_schema: analytics_staging
        schema: analytics_models
        table: task_autosql
      ddl:
        indexes:
          primary_key:
            columns:
              - column1
              - column2
          idx1:
            columns:
              - column1
      permissions:
        role_name: SELECT
    ...
    ```
