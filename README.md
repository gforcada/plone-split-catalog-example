# How to use a split ZODB mount point for the portal_catalog

Example buildout project where the `portal_catalog` is split on its own database ZODB database file.

This is based on a [thread from 2006 on plone_users mailing list](https://sourceforge.net/p/plone/mailman/plone-users/thread/77be04730612180735we69c8ffibeeb3072ff553a3b%40mail.gmail.com/),
see also [another source](https://www.plone-entwicklerhandbuch.de/produktivserver/performance/zcatalog/katalog-in-eigener-zodb)
and [Plone community feedback](https://community.plone.org/t/split-the-portal-catalog-from-data-fs-on-its-own-storage/15412).

## Why?

Basically for performance reasons.
The `portal_catalog` is used much more than the rest of the content on the ZODB.
Being able to have different cache settings for each database individually can improve performance a lot.

But more importantly,
because I (Gil Forcada) have a website with this set up already working
and I can not reproduce it, hence I'm creating this separate small project to try to replicate,
and document!, how to achieve it.

## How to
The idea is rather simple (though it gets complex as you will see):
- create a Plone site on it
- rename the database to something else
- hook up that database on another Plone instance

That's the theory, there are much more steps involved though.

## In depth
Let's go step by step and see where the problem is though.

To try it yourself, please clone this repository:

```shell
$ git clone git@github.com:gforcada/plone-split-catalog-example
$ cd plone-split-catalog-example
$ python3.8 -m venv venv
$ . venv/bin/activate
$ pip install -r requirements.txt
$ buildout
```

You should have a `bin` folder with a `zeo` and `instance` scripts.

In two different terminals run them:

```shell
$ ./bin/zeo fg
$ ./bin/instance fg
```

Now head over to your browser of choice and point it to http://localhost:8080

### Create the first plone site

Click on the button to create a new plone instance,
authenticate yourself as `admin` and `admin` password (as defined on `buildout.cfg`).

Once it finishes, go to the ZMI,
i.e. http://localhost:8080/Plone/manage_main and **delete** all objects but the `portal_catalog`.

### Pack and rename

Next, stop `Ctrl+C` the plone instance,
i.e. where you are running `./bin/instance fg`. Keep ZEO (`./bin/zeo fg`) running though!

On that terminal run:

```shell
$ ./bin/zeopack
```

Now, stop also ZEO with `Ctrl+C.

Move the `Data.fs` to `Catalog.fs`:

```shell
$ mv var/filestorage/Data.fs var/filestorage/Catalog.fs
```

Do some clean up:

```shell
$ rm -f var/filestorage/Data.fs.*
$ rm -f var/filestorage/Catalog.fs.*
```

Remove the `parts` folder and re-create it, otherwise the `admin` user does not work (no idea why):

```shell
$ rm -rf parts
$ buildout
```

### Final plone instance
Now we are on the final step, create a new plone instance and attach the previous one (now on `Catalog.fs`) to it.

For that, start again both ZEO and plone in two different terminals:

```shell
$ ./bin/zeo fg
$ ./bin/instance fg
```

Head over to http://localhost:8080 and create a new plone instance.

### Add the new mount point
Go to Plone's ZMI: http://localhost:8080/Plone/manage_main

This time though, rather than removing everything but `portal_catalog`, do **remove** `portal_catalog`.

Then on the ZMI top toolbar, select `ZODB Mount Point` on the `Select type to add` dropdown.

On the pop-up the `/Plone/portal_catalog` mount point should be already selected,
click on the `Create selected mount points` button, and a new `portal_catalog` should appear.

Still on the ZMI,
go to the new `portal_catalog` and on the `Advanced` tab click on the `Clear and Rebuild` button.

### Status
Although everything seems to look fine,
the `Catalog` tab on `portal_catalog` says that there is no content
and clearing and rebuilding the catalog does nothing for that to change.

On the other hand,
one can happily create content on plone and find it via the search form.

More over, looking at the `Data.fs` and `Catalog.fs`,
one sees that `Data.fs` keeps growing and shrinking when we add content,
remove content and pack the database, but `Catalog.fs` remains always at the same size.

**What's going on?!**

**Which are the wrong steps?!**

Any hints are highly appreciated!
