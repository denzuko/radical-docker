# radical-docker
Collection of docker services spun up with make(1)

Each folder one can spin up into its own compose namespace by executing:

```
make -C $folder_name all
```

folder | description
- | -
secbot | Experiments in launching bitlbee, znc, and eggdrop/pyirc bots. Client side can be anything thanks to bitlbee.
