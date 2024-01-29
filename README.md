# radical-docker
Collection of docker services spun up with make(1)

The source of truth for `docker-compose(1)` is the multiline variable in the makefile. This is
Here to help with configuration generation as Make *is* a templating language.

Further more each folder should have a manpage include. Read this with:

```
make -C $folder_name help
```

For each folder one can spin up into its own compose namespace by executing:

```
make -C $folder_name all
```

## Services

| folder | description |
| - | - |
| secbot | Experiments in launching bitlbee, znc, and eggdrop/pyirc bots. Client side can be anything thanks to bitlbee. |

