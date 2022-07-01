# How to use a split ZODB mount point for the portal_catalog

Example buildout project where the `portal_catalog` is split on its own database ZODB database file.

This is based on a [thread from 2006 on plone_users mailing list](https://sourceforge.net/p/plone/mailman/plone-users/thread/77be04730612180735we69c8ffibeeb3072ff553a3b%40mail.gmail.com/).

## Why?

Basically for performance reasons, the `portal_catalog` is used much more than the rest of the content on the ZODB.
Being able to have different cache settings for each database individually can improve performance a lot.

But more importantly, because I (Gil Forcada) have a website with this set up already working and I can not reproduce it,
hence I'm creating this separate small project to try to replicate, and document!, how to achieve it.

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
$ python3 -m venv venv
$ . venv/bin/activate
$ pip install zc.buildout
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

Click on the button to create a new plone instance, authenticate yourself as `admin` and `admin` password (as defined on `buildout.cfg`).

Once it finishes, go to the ZMI, i.e. http://localhost:8080/Plone/manage_main and **delete** all objects but the `portal_catalog` one.
