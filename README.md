# notify-osd-deb

minos' change on Ubuntu's own notify-osd package.

From 26.04 Ubuntu takes notify-osd from Debian, whose `debian/rules` deletes the D-Bus activation file
`/usr/share/dbus-1/services/org.freedesktop.Notifications.service` after `dh_install` (Debian #640120).
Without it nothing on the session bus can start the daemon, so `notify-send` fails with `ServiceUnknown`
unless notify-osd is already running. Ubuntu's own packaging, up to 24.04, shipped the file. minos keeps it:
the one commit drops the rule that deletes it, so the file upstream generates ships again and `notify-send`
starts notify-osd on demand, as on 22.04 and 24.04.

The change is to `debian/rules`, not a patch in `debian/patches/`: the file is removed by the packaging, not
by the program, so there is nothing for a quilt patch to change.

## Branches

One branch per Ubuntu release that needs the change: `resolute` (26.04). Ubuntu 22.04 and 24.04 ship the
file and need nothing.

Each one is that release's `ubuntu/<series>-devel` branch from Ubuntu's git of the package
(https://git.launchpad.net/ubuntu/+source/notify-osd), with its full history, plus one commit that removes
the `execute_after_dh_install` target of `debian/rules` (its `rm` and `rmdir` lines). Nothing else
changes, not even `debian/changelog`.

## How it is built

One Launchpad recipe per release, building into the minos PPA (`ppa:minos-archive/main`), starts from
Ubuntu's branch and merges this one:

    # git-build-recipe format 0.4 deb-version {debversion}build0+minos{time}
    lp:ubuntu/+source/notify-osd ubuntu/resolute-devel
    merge minos lp:~minos-archive/minos/+git/notify-osd-deb resolute

The version is Ubuntu's followed by `build0+minos` and the time of the build, so every build gets a new one:
a version taken from the branches alone repeats when the same commits are built twice, and the PPA refuses
the second upload, whose changelog entry carries another date. Ubuntu's
version ends in a digit (`…-3`), and a bare `+minos` there would sort above Ubuntu's next rebuild (`…-3build1`)
or update (`…-3ubuntu0.1`); `build0` keeps the minos version just above Ubuntu's current one and below any
newer Ubuntu upload. The recipes build daily when a branch they read has changed, so a new Ubuntu upload is
rebuilt with the change; if that build fails, Ubuntu's newer package wins over the old minos one.

## When Ubuntu uploads a new version

Rebase the one commit onto Ubuntu's branch and push it:

    git remote add ubuntu https://git.launchpad.net/ubuntu/+source/notify-osd
    git fetch ubuntu
    git checkout resolute
    git rebase ubuntu/ubuntu/resolute-devel
    git push --force-with-lease origin resolute

If the new upload ships the activation file again, the branch and its recipe are no longer needed. If its
version no longer ends in a digit (`…-3ubuntu1`), the recipe's `build0` goes. A new Ubuntu release that still
deletes the file gets its own branch, made the same way, and its own recipe.
