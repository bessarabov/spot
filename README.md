# SimploTask

SimploTask (aka `spot`) is a powerful and easy-to-use tool for effortless deployment and configuration management. It allows users to define a playbook with the list of tasks and targets, where each task consists of a series of commands that can be executed on remote hosts concurrently. SimploTask supports running scripts, copying files, syncing directories, and deleting files or directories, as well as custom inventory files or inventory URLs.

## Features

- Define tasks with a list of commands.
- Support for remote hosts specified directly or through inventory files/URLs.
- Everything can be defined in a simple YAML file.
- Run scripts on remote hosts.
- Built-in commands: copy, sync, and delete.
- Concurrent execution of task on multiple hosts.
- Customizable environment variables.
- Ability to override list of destination hosts, ssh username and ssh key file.
- Skip or execute only specific commands.
- Debug mode to print out the commands to be executed, output of the commands, and all the errors.
- A single binary with no dependencies.


## Options

SimploTask supports the following command-line options:

- `-f`, `--file=`: Specifies the playbook file to be used. Defaults to spot.yml. You can also set the environment
  variable $SPOT_FILE to define the playbook file path.
- `t`, `--task=`: Specifies the task name to execute. The task should be defined in the playbook file.
  If not specified all the tasks will be executed.
- `-d`, `--target=`: Specifies the target name to use for the task execution. The target should be defined in the playbook file and can represent remote hosts, inventory files, or inventory URLs. User can pass a host name or IP instead of the target name for a quick override. If not specified the default target will be used.
- `-c`, `--concurrent=`: Sets the number of concurrent tasks to execute. Defaults to 1, which means tasks will be executed sequentially.
  `-h, --host=`: Specifies the destination host(s) for the task execution. Overrides the host(s) defined in the playbook file
  for the specified target. Provide the `-h` flag multiple times with different hosts names or ips to set multiple destination hosts,
  e.g., `-h example1.com -h example2.com`
- `--inventory-fil=`: Specifies the inventory file to use for the task execution. Overrides the inventory file defined in the
  playbook file for the specified target.
- `--inventory-url=`: Specifies the inventory HTTP URL to use for the task execution. Overrides the inventory URL defined in the
  playbook file for the specified target.
- `-u`, `--user=`: Specifies the SSH user to use when connecting to remote hosts. Overrides the user defined in the playbook file
  for the specified task.
- `-k`, `--key=`: Specifies the SSH key to use when connecting to remote hosts. Overrides the key defined in the playbook file
  for the specified task.
- `-s`, `--skip=`: Skips the specified commands during the task execution. Provide a comma-separated list of command names to skip.
- `-o`, `--only=`: Runs only the specified commands during the task execution. Provide a comma-separated list of command names to execute.
- `--dbg`: Enables debug mode, providing more detailed output and error messages during the task execution.
- `-h`, `--help`: Displays the help message, listing all available command-line options.

## Example Playbook

```yml
targets:
  prod:
    hosts: ["h1.example.com", "h2.example.com"]
  staging:
    inventory_file: "testdata/inventory"
  dev:
    inventory_url: "http://localhost:8080/inventory"

tasks:
  deploy-things:
    commands:
      - name: wait
        script: sleep 5s
      - name: copy configuration
        copy: {"src": "testdata/conf.yml", "dst": "/tmp/conf.yml", "mkdir": true}
      - name: sync things
        sync: {"src": "testdata", "dst": "/tmp/things"}
      - name: some command
        script: |
          ls -laR /tmp
          du -hcs /srv
          cat /tmp/conf.yml
          echo all good, 123
      - name: delete things
        delete: {"loc": "/tmp/things", "recur": true}
      - name: show content
        script: ls -laR /tmp

  docker:
    commands:
      - name: docker pull and start
        script: |
          docker pull umputun/remark42:latest
          docker stop remark42 || true
          docker rm remark42 || true
          docker run -d --name remark42 -p 8080:8080 umputun/remark42:latest
        env: {FOO: bar, BAR: qux}
```

## Runtime variables

SimploTask supports runtime variables that can be used in the playbook file. The following variables are supported:

- `{SPOT_REMOTE_HOST}`: The remote host name or IP address.
- `{SPOT_REMOTE_USER}`: The remote user name.
- `{SPOT_COMMAND}`: The command name.
- `{SPOT_TASK}`: The task name.

Variables can be used in the following places: `script`, `copy`, `sync`, `delete` and `env`, for example:

```yml
tasks:
  deploy-things:
    commands:
      - name: copy configuration
        copy: {"src": "{SPOT_REMOTE_HOST}/conf.yml", "dst": "/tmp/conf.yml", "mkdir": true}
      - name: sync things
        sync: {"src": "testdata", "dst": "/tmp/{SPOT_TASK}/things"}
      - name: some command
        script: |
          ls -laR /tmp/${SPOT_COMMAND}
        env: { FOO: bar, BAR: "{SPOT_COMMAND}-blah" }
      - name: delete things
        delete: {"loc": "/tmp/things/{SPOT_REMOTE_USER}", "recur": true}

```


## Getting Started

- Install SimploTask by running `go install github.com/umputun/simplotask` or download the latest release from the Releases page.
- Create a configuration file, as shown in the example above, and save it as config.yml.
-  Run SimploTask using the following command: `spot`. This will execute all the tasks defined in the default `spot.yml` file for the `default` target with a concurrency of 1.
- To execute a specific task, use the `-t` flag: `spot -t deploy-things`. This will execute only the `deploy-things` task.
- To execute a specific task for a specific target, use the `-t` and `-d` flags: `spot -t deploy-things -d prod`. This will execute only the `deploy-things` task for the `prod` target.


## Why SimploTask?

SimploTask is designed to provide a simple, efficient, and flexible solution for deployment and configuration management. 
It addresses the need for a tool that is easy to set up and use, while still offering powerful features for managing infrastructure.
Below are some of the reasons why you should consider using SimploTask:

1. **Simplicity**: SimploTask's primary goal is to be as simple as possible without sacrificing functionality. Its configuration is written in YAML, making it easy to read and understand. You can quickly create and manage tasks, targets, and commands without dealing with complex structures or concepts.
2. **Flexibility**: SimploTask is designed to be flexible and adaptable to various deployment and configuration scenarios. You can use it to manage different targets, such as production, staging, and development environments. It supports executing tasks on remote hosts directly or through inventory files and URLs, allowing you to use existing inventory management solutions.
3. **Extensibility**: SimploTask is built to be extensible, allowing you to define custom scripts for execution on remote hosts, as well as offering built-in commands for common operations such as copy, sync, and delete. This extensibility enables you to create complex workflows for deployment and configuration management, tailored to your specific needs.
4. **Concurrent Execution**: SimploTask supports concurrent execution of tasks, allowing you to speed up the deployment and configuration processes by running multiple tasks simultaneously. This can be particularly helpful when managing large-scale infrastructure or when time is of the essence.
5. **Customizable**: SimploTask provides various command-line options and environment variables that enable you to customize its behavior according to your requirements. You can easily modify the playbook file, task, target, and other parameters, as well as control the execution flow by skipping or running specific commands.
6. **Lightweight**: SimploTask is a lightweight tool, written in Go, that does not require heavy dependencies or a complex setup process. It can be easily installed and run on various platforms, making it an ideal choice for teams looking for a low-overhead solution for deployment and configuration management.

In conclusion, SimploTask is a powerful and easy-to-use tool that simplifies the process of deployment and configuration management while offering the flexibility and extensibility needed to cater to various use cases. If you value simplicity, efficiency, and a customizable experience, SimploTask is a great choice for your infrastructure management needs.


## Is it replacing Ansible?

SimploTask is not intended to be a direct replacement for Ansible. While both tools can be used for deployment and configuration management, there are some key differences between them:

- **Complexity**: Ansible is a more feature-rich and mature tool, offering a wide range of modules and plugins that can automate many different aspects of infrastructure management. SimploTask, on the other hand, is designed to be simple and lightweight, focusing on a few core features to streamline the deployment and configuration process.
- **Learning Curve**: Due to its simplicity, SimploTask has a lower learning curve compared to Ansible. It's easier to get started with SimploTask, making it more suitable for smaller projects or teams with limited experience in infrastructure automation. Ansible, while more powerful, can be more complex to learn and configure, especially for newcomers. 
- **Customization**: While both tools offer customization options, Ansible has a more extensive set of built-in modules and plugins that can handle a wide range of tasks out-of-the-box. SimploTask, in contrast, relies on custom scripts and a limited set of built-in commands for its functionality, which might require more manual configuration and scripting for certain use cases.
- **Community and Ecosystem**: Ansible has a large and active community, as well as a vast ecosystem of roles, modules, and integrations. This can be beneficial when dealing with common tasks or integrating with third-party systems. SimploTask, being a smaller and simpler tool, doesn't have the same level of community support or ecosystem.
- **Ease of installation and external dependencies**: One of the most significant benefits of SimploTask is that it has no dependencies. Being written in Go, it is compiled into a single binary that can be easily distributed and executed on various platforms. This eliminates the need to install or manage any additional software, libraries, or dependencies to use SimploTask. Ansible, on the other hand, is written in Python and requires Python to be installed on both the control host (where Ansible is run) and the managed nodes (remote hosts being managed). Additionally, Ansible depends on several Python libraries, which need to be installed and maintained on the control host. Some Ansible modules may also require specific libraries or packages to be installed on the managed nodes, adding to the complexity of managing dependencies.

SimploTask can be a good choice if you're looking for a lightweight, simple, and easy-to-use tool for deployment and configuration management, particularly for smaller projects or when you don't need the extensive features offered by Ansible. However, if you require a more comprehensive solution with a wide range of built-in modules, plugins, and integrations, Ansible might be a better fit for your needs. 

The simplicity of SimploTask's single binary distribution and lack of dependencies make it an attractive choice for teams who want a lightweight, easy-to-install, and low-maintenance solution for deployment and configuration management. While Ansible offers more advanced features and a comprehensive ecosystem, its dependency on Python and additional libraries can be a hurdle for some users, particularly in environments with strict control over software installations or limited resources.


## Contributing

Please feel free to submit issues, fork the repository, and send pull requests.

## License

This project is licensed under the MIT License. See the LICENSE file for more information.
