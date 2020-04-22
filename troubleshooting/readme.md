1. to troubleshot the docker container, if we can access the node the container run
  - get the PID of the process of the container on the host 
  - enter into the namespace of the container
    ```
    nsenter -t $PID --uts --ipc --net --pid
    ```
    or just enter into perticular namespace, for example network namespace
    ```
    nsenter -t $PID -n
    ```

      ```
      nsenter options:
      -a, --all              enter all namespaces
      -t, --target <pid>     target process to get namespaces from
      -m, --mount[=<file>]   enter mount namespace
      -u, --uts[=<file>]     enter UTS namespace (hostname etc)
      -i, --ipc[=<file>]     enter System V IPC namespace
      -n, --net[=<file>]     enter network namespace
      -p, --pid[=<file>]     enter pid namespace
      -C, --cgroup[=<file>]  enter cgroup namespace
      -U, --user[=<file>]    enter user namespace
      -S, --setuid <uid>     set uid in entered namespace
      -G, --setgid <gid>     set gid in entered namespace
          --preserve-credentials do not touch uids or gids
      -r, --root[=<dir>]     set the root directory
      -w, --wd[=<dir>]       set the working directory
      -F, --no-fork          do not fork before exec'ing <program>
      -Z, --follow-context   set SELinux context according to --target PID
      ```
  - run the commands to further research, as we running the host commands inside the container namespace, so all host commands can run here.
    ```
    curl google.com
    ```

