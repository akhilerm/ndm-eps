# Proposed Refactoring for NDM

### Principles followed 

1. We will be following an Onion architecture. `BlockDevice` will be the core of NDM
2. All high level components that are visible outside for NDM, will come in the root directory, NOT a `pkg`
3. All directories in `pkg` should be independent of each other except `pkg/apis` and `pkg/controller`. Except these two,
  all packages should depend only on go builtin packages or vendored packages.
4. Declare interfaces to closer where they are used.
5. `cmd` directory should contain only entrypoint for the binaries from this repo.

```
node-disk-manager/
├── agentdaemon
├── blockdevice
│   └── blockdevice.go
├── cmd
│   ├── agent-daemon
│   │   └── main.go
│   ├── exporter
│   │   └── main.go
│   └── manager
│       └── main.go
├── config
├── db
│   └── kubernetes
│       ├── store.go
│       └── upgrade.go
├── eventhandler
│   └── eventhandler.go
├── filters
│   ├── mount-filter
│   ├── path-filter
│   └── property-filter
├── pkg
│   ├── apis
│   │   ├── blockdevicelcaim_types.go
│   │   ├── blockdevice_types.go
│   │   └── disk_types.go
│   ├── controller
│   │   ├── blockdeviceclaim-controller
│   │   ├── blockdevice-controller
│   │   └── disk-controller
│   ├── mount
│   │   ├── monitor.go
│   │   └── mount.go
│   ├── seachest
│   ├── smart
│   └── udev
│       ├── device.go
│       └── monitor.go
└── probes
    ├── capacity-probe
    ├── seachest-probe
    ├── smart-probe
    └── udev-probe

```

# Concepts Used

#### BlockDevice
`BlockDevice` will be the core of the project, which will be passed around similar to a go `context`.
```
type BlockDevice struct {
	UUID
	DevPath
	MountPoint
	FileSystem
	....
}
```

#### cmd
The `cmd` directory at the root of the project will contain packages for each binary that can be generated from this repository. 
It should contain only Cobra CLI command code, or the main() for the function

#### AgentDaemon
Agent daemon is the main ndm daemon process that will be running on the system. The agent daemon will be started by a 
call from `cmd/agentdaemon`. 
Following will be done
1. Load the configuration
2. Initialize DB client
3. Initialize probes and filters based on config
4. Start the listeners/monitor based on config
5. Transfer the control to eventhandler

#### EventHandler
An eventhandler is started by the agent daemon. 

Whenever a system event is generated like new device attached or a change happened in the mount tree, the listeners for those
events will generate a minimal `BlockDevice` struct and send it to a channel (say `EventMessage` ) for processing.

The eventhandler will be receiving events from `EventMessage` channel and whenever and process it. The processing will involve
filling details in the minimal `BlockDevice` struct using probes, applying filters and performing a DB operation.

**NOTE**: Regardless of the listener from which the event is generated, `pkg/udev` will be used to generate a UUID for the BlockDevice 

#### Filters
Filter will have all the filters that we use in NDM. Every filter at its core will use a `property-filter`
eg: - `path-filter` (filter based on device path, `/dev/sda`, `/dev/loop*`)will be converted to a `property-filter` 
	which uses device path as the property
    - `mount-filter` (filter based on mountpoint, `/`, `/home`, `/var`, `/mnt`) will be converted to a `path-filter`
    by getting the devicepath from mount path.

This approach will allow users to create their own filters without requiring any code change. They can use any property
listed in `udevadm info` and create the config.

```
--> MountFilter------
                     \
                      \
                       \
                        \
----------------------> PathFilter ------------> PropertyFilter
                                                 /
                                                /
--> AnyUserDefinedFilter....--------------------
    	(eg: Vendor)
```

The property filter struct will contain something like:
```
type Property string
type PropertyValue []string

type PropertyFilter struct {
	Include map[Property]PropertyValue
	Exclude map[Property]PropertyValue
}
```
#### Probes
Probes are used to fetch various details of the disk from the system. SMART, Seachest, Capacity, Mount, Udev will be the various probes
used to fetch the data.

#### Config
Config will be used for parsing the user provided configuration into an internal struct which can be used for initializing probes, 
filters, system monitors, and DB clients.

#### Install
// TODO @kmova @akhilerm
How to manage installation of the components, mainly CRDs. This is something that is to be done by manager
