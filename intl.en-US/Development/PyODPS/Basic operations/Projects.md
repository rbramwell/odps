---
keyword: [project, get\_project\(\)]
---

# Projects

This topic describes how to perform basic operations on projects by using PyODPS.

A project is the basic organizational unit of MaxCompute. For more information, see [Project](/intl.en-US/Product Introduction/Definitions/Project.md).

You can perform the following basic operations on a project:

-   Obtain a project: You can call the `get_project()` method of a MaxCompute entry object to obtain a specific project.

    ```
    project = o.get_project('my_project')  # Obtain the specified project.
    project = o.get_project()              # Obtain the current project.
    ```

    To obtain a specific project, you must set the project\_name parameter to the name of the project for the get\_project\(\) method. If you do not specify a project name for the method, the method returns the current project.

-   Check whether a project exists: You can call the `exist_project()` method to check whether a specific project exists.

