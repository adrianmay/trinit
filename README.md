This is a ludicrously simple init script system.

The philosophy is:

* "B is dependent on A" means that for all of the time that B is starting, running or stopping, A must be fully operational.
* Use a directory tree to encode dependencies between services (subdirectories depend on their parent directories.)
* Fully start a service before attempting to start any of its dependents.
* Fully stop a service before attempting to stop any of its dependencies.
* Independently and concurrently bring subtrees of a given directory up and down. This replaces runlevels, e.g., the root of the tree is the emergency run level.
* Use scripts for everything and minimise internal magic (i.e. more like runit than systemd.)
* Services are defined by `start` and `stop` scripts which block until the service is fully up or down.
* Global `up` and `down` scripts recursively run the `start` and `stop` scripts, recording state under `/tmp`.
* There's no pid 1 process yet but trinit can be installed in some other init system as a run level. Runit's pid 1 process is very simple and fits nicely for trinit. Just make a runit job that `run`s and `finish`es with `up .` and `down .` respectively.

I wrote this cos runit is the only init system simple enough for my feeble brain but it unleashes chaos during shutdown.

If a service has multiple dependencies, then one of them is a parent directory as usual, and you make symlinks for the others (like `eg/a/a1/a32`.) Additionally, the directory for the multiply-dependent service needs a `deps` file listing all the dependencies, including the parent directory. Starting of the multiply-dependent service is happily aborted ("postponed") unless all the services in `deps` are up, i.e., only in response to the starting of the last dependency will it proceed. It's a hassle to have to enter multiple dependencies as both a symlink and a line in `deps` but at least there's a consistency checker to make sure anthing mentioned in `deps` is either the parent directory of has a corresponding symlink. In future there might be an automatic way to populate `deps` (in which case it should live under /tmp.)

It shouldn't attempt to be a process monitor, but there are solutions for that (e.g. `while`) that could be invoked in a `start` script.

### Installation

For a runit system, copy this repo's runit/trinit to sv and link to it in the usual way. Note the `sleep infinity` in `run` which converts from runit's "everything must be a process so I can monitor it" philosophy to trinit's "the start script must exit to tell me to proceed to the children".

For other init systems like systemd, do something equivalent.

/tmp should actually be a tmpfs mount so it's sure to be empty upon boot.
