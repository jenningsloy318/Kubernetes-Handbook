# Journald 

Modern Linux are leverage journald for logs, but in default conf, journal logs are not persisted, we need make it happend.

## Configure journald

1. Create neccessary directory
    ```
    mkdir  /var/log/journal
    ```
2. Modify the parameters in  `/etc/systemd/journald.conf`.
    ```
    [Journal]
    Storage=auto
    #Compress=yes
    #Seal=yes
    #SplitMode=uid
    #SyncIntervalSec=5m
    #RateLimitInterval=30s
    #RateLimitBurst=1000
    SystemMaxUse=20%
    #SystemKeepFree=
    #SystemMaxFileSize=
    #SystemMaxFiles=100
    #RuntimeMaxUse=20
    #RuntimeKeepFree=
    #RuntimeMaxFileSize=
    #RuntimeMaxFiles=100
    #MaxRetentionSec=
    #MaxFileSec=1month
    #ForwardToSyslog=yes
    #ForwardToKMsg=no
    #ForwardToConsole=no
    #ForwardToWall=yes
    #TTYPath=/dev/console
    #MaxLevelStore=debug
    #MaxLevelSyslog=debug
    #MaxLevelKMsg=notice
    #MaxLevelConsole=info
    #MaxLevelWall=emerg
    ```

    - Storage=auto：“auto”: The storage mode is like persistent — data will be written to disk under /var/log/journal/
    - SystemMaxuse: This parameter controls the maximum disk space the journal can consume when it’s persistent
    - ### Make sure the dir /var/log/journal exists
    - For all parameter, refer to https://www.freedesktop.org/software/systemd/man/journald.conf.html
