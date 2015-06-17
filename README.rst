============
Git Workshop
============


Warum und überhaupt?
----------------------

Es hat sich durchgesetzt und geht nicht mehr weg.
Die meisten Workflows lassen sich auf alle anderen DVCS und auch auf SVN übertragen. Andere VCS tun anders weh.
Aber hey. Wir haben git.

Vorteil für mich:

* Man kann komplett lokal arbeiten.
* Merges testen, etc.
* Zerschießt das Hauptrepo nicht



Dezentral?

Jeder hat das komplette Repo lokal auf der Platte.
In echt gibt es aber natürlich ein Master-Repo. z.B. gitlab/github

config the global Stuff
-----------------------

username und email:

.. code-block:: sh

   git config --global user.name "Christoph Glaubitz"
   git config --global user.email "me@example.net"
   cat ~/.gitconfig

Wichtig, weil dort alles stehen kann. z.B. wenn man auf einem Host als root committed.
Authentifizierung an github/gitlab erfolgt über ssh-key, unabhängig davon was in den Commits steht. Des weiteren gibt es noch die Möglichkeit Commits zu signieren (PGP) und die Signatur zu verifizieren. Darauf möchte ich aber nicht näher eingehen.

Benutzung, die Basics
---------------------

Repo herunterladen:

.. code-block:: sh

   git clone git@gitlab.example.net:cglaubitz/git-workshop.git

Dateien anlegen/editieren:

.. code-block:: text

   ...

Den Status prüfen und ein commit von allem

.. code-block:: sh

   git status
   git commit -a # alles

oder gezielte Dateien

.. code-block:: sh

   git add testfile
   git commit

Die History prüfen

.. code-block:: sh

   git log

Nun kann man noch nachträgliche Änderungen durchführen. Z.B. wenn Name und E-Mail falsch konfigurieirt waren.

.. code-block:: sh

   git config --global user.name "Christoph Glaubitz"
   git config --global user.email "c.glaubitz@example.net"
   
   git commit --amend --reset-author

In sehr komplizierter Art und Weise kann man dies auch für Beliebige commits in der Vergangenheit machen. Allerdings gilt, dadurch ändert sich die gesamget History! (später mehr)

und hoch

.. code-block:: sh

   git push

Manpages:

.. code-block:: sh

   man git-COMMAND
   # z.B.:
   man git-push

Tipps:


zusätzliche aliases (git lg)

.. code-block:: text

   git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"

   git lg
   ...


liquidprompt [14]


Als Erweiterung des shell-Promptes. Zeigt u.a. den aktuellen Branch und die Anzahl der Änderungen an. Liquidprompt ist dabei nicht auf git beschränkt.

Branches
--------
Git wurde dafür Entworfen um das Arbeiten und Wegwerfen von Trees/Branches so einfach wie möglich zu machen.
Ein großer Vorteil gegenüber SVN, da dort branches und tags nur über die Konvention beim anlegen des Repos existieren.
repo/trunk, branches, tags
Schwierig nachträglich einzuführen.

Branch erzeugen (wechselt auch gleich)

.. code-block:: sh

   git checkout -b BRANCH [ABSPRUNG_BRANCH]

lässt man ABSPRUNG_BRANCH weg, springt man vom aktuellen Branch ab.

Einfaches Wechseln der Branches

.. code-block:: sh

   git checkout BRANCHNAME


Aktueller Pfad bleibt der selbe. Es brauchen also keine Buildprozesse angepasst werden wenn ein anderer branch gebaut werden muss.
Zeigen anhand von syseleven-puppet-dev-Environment.

.. code-block:: sh

   switchpuppetdev pp-XXX
   git checkout trunk
   git checkout -b merge-trunk-pp-XXX-test
   # Hätte den selben Effekt:
   git checkout -b merge-trunk-pp-XXX-test trunk
   git merge pp-XXX

Bis hierhin ist das alles lokal in meinem Repo. Kein anderer ist betroffen.

.. code-block:: sh

   really_run_puppet_agent -f

auf dem einem entsprechenden Host ist jetzt auf trunk + meinen Änderungen. Kein Vezeichnis oder Environment oder was auch immer musste angepasst werden.
Integration des Feature-Branches und des aktuellen trunk/master kann einfach getestet werden.

Merge / Rebase
--------------

Szenario:
Es gibt zwei parallel laufende Äste, die den selben Ursprung haben.

Bei einem Merge werden diese oben wieder zusammen gebunden.
Bei einem Rebase wird einer der beiden abgesägt und auf den anderen aufgesetzt.

Merge:

.. code-block:: text

   *
   M \
   M  F
   M  F
   M /
   *

Rebase:

.. code-block:: text
   
               F
               F
   M  F        M
   M  F  ===>  M
   M /         M
   *           *

Die Commit-IDs von M bleiben erhalten, die von F ändern sich.
D.h. äußerste Vorsicht bei Rebase.
Mit einem Rebase verändert man quasi die Vergangenheit! [4]
Es gilt:
Kein rebase nach einer Veröffentlichung.
Ansonsten kann es passieren, dass andere schon von einem commit abgesprungen ist, den es in der Form nicht mehr gibt.

git erkennt dies auch und zwingt einen beim push zu

.. code-block:: sh

   git push --force

Rebase also nur wenn man weiß was man tut.

Konfiguration des lokalen Repositories
--------------------------------------

.. code-block:: sh

   .git/config
 
Tags, branches und Remotes sind nur "Pointer" auf hashes

.. code-block:: sh

   .git/refs


Objekte unter .git/objects siehe [3]

Hooks
-----

Befinden sich in .git/hooks
Scripte die von git zu diversen Events aufgerufen werden.
z.B. um eine Aktion vor dem commit durchzuführen:

.. code-block:: sh

   .git/hooks/pre-commit


Beendet dieser mit exit 0, wird der commit durchgeführt.
Exit != 0 verhindert den Commit.
Dies kann z.B. sinnvoll sein um zu prüfen ob "zu große" Dateien hochgeladen werden.
Oder einen linter über geänderten Dateien laufen zu lassen und den Commit nur zu machen wenn dieser keine Fehler wirft.
Unter .git/hooks/ gibt es für jedes Event ein Beispiel.

Workflows [1,7]
---------------

* Centralized
* Feature Branch
* Gitflow
* Forking

Kurz. Es gibt viele Möglichkeiten git zu verwenden. Man sollte sich in der Gruppe zu einer Variante entscheiden, die alle benutzen.

Meiner persönlichen Ansicht nach, ist es empfehlenswert eine geradlinige History zu haben. Also so wenn immer möglich Fast Forward zu mergen. Dafür muss bei Commit-Nachrichten Disziplin eingehalten und die Ticket-id mit angegeben werden.

Das folgende Beispiel mit Issue-1...

.. code-block:: text

   Master   1 2 3 - - -
                 \
   Issue-1        4 5 6

   git log
   commit 4
   Author: Christoph Glaubitz <c.glaubitz@example.net>
   Date:   ...

       Issue-1 did some stuff #1

       ...
   commit 5
   Author: Christoph Glaubitz <c.glaubitz@example.net>
   Date:   ...

       Issue-1 did some stuff #2

       ...
   commit 5
   Author: Christoph Glaubitz <c.glaubitz@example.net>
   Date:   ...

       Issue-1 did some stuff #3

       ...

... sollte Fast Forward gemerged werden:

.. code-block:: text

   git merge Issue-1

Das Resultat:

.. code-block:: text

   Master   1 2 3 4 5 6

   git lg

   * 6 (HEAD, master) Issue-1 did some stuff #3 (1 hour ago) Christoph Glaubitz <c.glaubitz@example.net>
   * 5 Issue-1 did some stuff #2 (1 hour ago) Christoph Glaubitz <c.glaubitz@example.net>
   * 4 Issue-1 did some stuff #1 (1 hour ago) Christoph Glaubitz <c.glaubitz@example.net>
   * 3 Issue-0 ... Some Other Guy <s.o.g@example.net>
   * 2 Issue-0 ... Some Other Guy <s.o.g@example.net>
   * 1 Issue-0 ... Some Other Guy <s.o.g@example.net>

Unter [2] ist ein Artikel zur Diskussion von Gitflow verlinkt.


Merge / Rebase im Detail
------------------------

Gibt es nicht doch Situationen in denen rebase sinnvoll ist?

Rebase hat natürlich seine Daseinsberechtigung. Wie schon erwähnt wird damit die Vergangenheit verändert. Es lassen sich also z.B. einzelne commits ändern oder entfernen. Man muss eben nur darauf achten, rebases nur auf commits anzuwenden, die noch nicht Veröffentlicht sind.
Häufig verwendet man

.. code-block:: sh

   git rebase -i COMMIT_ID

um mehrere commits zu einem zusammen zu fassen. Im git-speech, squash commits.

Entwickelt man ein Feature, committed man wahrscheinlich relativ häufig. Oft sind aber nicht die einzelnen commits wichtig, sondern nur das Feature an sich.

.. code-block:: text
   
   Master   1 2 3 - - - -
                 \
   Feature        4 5 6 7

Commits 4 bis 7 sind einzeln eigentlich nicht relevant und können zu einem zusammengefasst werden. Dazu muss auf den Commit 3 rebased werden.

.. code-block:: text

   git rebase -i 3
   
   pick 4 feature1 - added file2                                             
   pick 5 feature1 - changed stuff in file1
   pick 6 feature1 - changed more stuff
   pick 7 feature1 - changed even more stuff

Wir behalten Commit 4, die restlichen nehmen wir mit in diesen.

.. code-block:: text

   pick 4 feature1 - added file2                                             
   squash 5 feature1 - changed stuff in file1
   squash 6 feature1 - changed more stuff
   squash 7 feature1 - changed even more stuff

Resultat:

.. code-block:: text

   Master : 1 2 3 -
                 \
   Feature:       4'

Bei jeder Veränderung, die durch rebase durchgeführt wird, gibt es eine neue Commit ID. Alle Commits von 4 - 7 sind verloren und durch 4' ersetzt.
Auf dem selben Weg lassen sich auch commit-Messages ändern.


Ein Weiterer Anwendungsfall für rebase ist sich die Arbeit aus dem originalen Branch in seinen branch zu ziehen. Dies ist vor allem bei lang laufenden Feature-Branches nötig.

Bsp:

.. code-block:: text

   Master : 1 2 3 - 8 9 - -
                  \
   Feature:        4 5 6 7

Ein rebase kann auf einen Vorfahren angewendet werden.

.. code-block:: sh

   git rebase master

resultiert in:

.. code-block:: text

   Master : 1 2 3 8 9 - - - - - -
                     \
   Feature:           4' 5' 6' 7'

Durch einen rebase des Feature-Branches vor dem Merge in master wird sichergestellt, dass ein fast-forward-merge möglich ist. [5]

Das Resultat eines

.. code-block:: text

   git checkout master
   git merge feature
   
   Master : 1 2 3 8 9 4' 5' 6' 7'


Das bringt und zu der Frage: Fast-Forward oder nicht? [2]

Fast-Forward bedeutet, dass nur der Pointer master weiter bewegt und kein neuer Commit erzeugt wird. Dies ist nur möglich wenn der Absprung des Feature-Branches auf den letzten Commit in Master zeigt.

.. code-block:: text

   Master : 1 2 3
                 \
   Feature:       4 5 6 7
   
   git checkout master
   git merge feature
   
   Master: 1 2 3 4 5 6 7

Sind in der Zwischenzeit Commits in Master passiert, ist ein Fast-Forward nicht mehr möglich. D.h. der merge resultiert in einen so genannten "Merge Commit" (Zeige in syseleven-base)

.. code-block:: text

   Master : 1 2 3 8
                 \
   Feature:       4 5 6 7
   
   git checkout master
   git merge
   
   Master : 1 2 3 8 - - - 9
                \       /
                 4 5 6 7

Der Commit 9 kann hier mit unter auch mit dem Beheben eines Konfliktes einher gehen.

Wie oben erwähnt, kann das durch ein vorheriges rebase verhindert werden.


Das Verhalten bei einem Merge kann man mit git merge --ff bzw. --no-ff steuern.

Es ist also auch möglich für jeden Merge einen "Merge Commit" erzeugen zu lassen. Auch wenn ein Fast-Forward möglich wäre.

.. code-block:: text

   Master : 1 2 3
                 \
   Feature:       4 5 6 7
   
   git checkout master
   git merge --no-ff feature
   
   Master : 1 2 3 - - - - 8
                 \       /
                  4 5 6 7

Es gibt Kontroversen ob man "Merge Commits" haben möchte oder nicht. Die einen Argumentieren damit, dass sich Feature-Branches sehen lassen, obwohl diese nach dem Merge gelöscht werden. Die andere Seite mag die überflüssigen Zusätzlichen Commits nicht, die in der Regel auch wenig Aussage treffen. In jedem Fall machen die zusätzlichen Commits die History schwerer zu lesen. Verzichtet man auf "Merge Commits" sollte man aber sehr darauf achten, Ticketnummern in die Überschriften der Commit Nachrichten zu schreiben. [2,8]

History flach halten
--------------------

Ausgehend davon, dass die History so schmal wie möglich gehalten werden soll, also wir mit Fast Forward mergen.


Wir haben aber schon festgestellt, dass Fast Forward nur funktioniert, wenn sich der Master während der Arbeit im Feature-Branch nicht verändert hat. Dies kriegen wir mit einem rebase im Feature-Branch für gewöhnlich in den Griff.


Als Beispiel aber mal zwei Feature-Branches, die beide zur gleichen Zeit gemerged werden sollen. Einer kann auf jeden Fall Fast Forward gemerged werden, der zweite aber nicht. Eine Lösung wäre den entsprechenden Entwickler nach dem ersten Merge rebasen zu lassen.

.. code-block:: text

   master-dev   : git merge feature-1
   master-dev   : git push
   feature-2-dev: git pull
   feature-2-dev: git checkout master
   feature-2-dev: git checkout feature-2
   feature-2-dev: git rebase master
   master-dev   : git pull
   master-dev   : git merge feature-2

Dies erfordert aber eine gute Portion koordination. master-dev sollte den rebase nicht selber machen, da feature-2-dev dann andere Commit-Ids hätte. Möchte master-dev die Arbeit ohne feature-2-dev erledigen, aber dessen Commit-Ids nicht verändern, gibt es weitere Möglichkeiten. Die einfachste wäre ein weiterer Zwischen-Branch.

.. code-block:: text

   master-dev   : git merge feature-1
   master-dev   : git branch feature-2-merge feature-2
   master-dev   : git checkout feature-2-merge
   master-dev   : git rebase master
   master-dev   : git merge feature-2-merge

Eine weitere Lösung ist ein cherry-pick der commits aus feature-2.

.. code-block:: text

   master-dev   : git merge feature-1
   master-dev   : git cherry-pick COMMIT_ID_OF_FEATURE_2_1 COMMIT_ID_OF_FEATURE_2_2 ...


In beiden Fällen darf man aber feature-2 nicht mergen, denn sowohl rebase als auch cherry-pick neue Commit-IDs erzeugen. D.h. Es gibt die selben Code-Änderungen jetzt unter verschiedenen IDs im Repo. Also Achtung.


Arbeiten mit Remotes
--------------------

Bisher haben wir eigentlich nur mit lokalen Repositories gearbeitet. Im Falle der Puppet-Entwicklung haben wir eine Remote-Lokation. Die erste Remote wird im git-jargon origin genannt.

Siehe .git/config bzw. git remote

.. code-block:: sh

   git remote -v
    origin git@gitlab.example.net:openstack/openstack-heattemplates.git (fetch)
    origin git@gitlab.example.net:openstack/openstack-heattemplates.git (push)

Habe ich lokal ein Repo angelegt, hat dieses überhaupt noch kein remote. Hinzufügen können wir das so:

.. code-block:: sh

   git remote add origin git@gitlab.example.net:cglaubitz/git-workshop.git
   git push -u origin master

-u bedeutet hier, dass ein tracking zwischen unserem masters und origin/master erstellt wird.

Es lassen sich mit git remote add beliebig viele weitere Remotes hinzufügen, deren Namen vaiabel sind. Aber warum sollte man das tun?

Siehe Forking-Workflow. Dieser kommt vor allem bei Projekten auf github vor.

Ein kleines Beispiel:

Wir möchten das Puppet-Modul puppet-corosync verwenden und stoßen an Grenzen. Es rechtfertig sich nicht ein eigenes Modul zu schreiben, wir möchten puppet-corosync erweitern und unsere Verbesserungen wieder zurück an puppetlabs liefen. Auch wenn wir mit Feature-Branches Arbeiten, werden die Puppetlabs-Leute uns keine Berechtigungen auf ihr Repo einrichten.


Um puppet-corosync zu verwenden haben wir einen Clone auf einem unserer Rechner angelegt.

Wir erzeugen auf github einen fork in den syseleven-Space

https://github.com/puppet-community/puppet-corosync -> fork

===>

https://github.com/syseleven/puppet-corosync


Wir können nun die Remotes unseres Clones anpassen. Zuerst richten wir den jetzigen origin als upstream ein.

.. code-block:: sh

   git remote add upstream https://github.com/puppet-community/puppet-corosync
   git branch -rl
     origin/HEAD -> origin/master
     origin/expected_votes
     origin/implement-version
     origin/master
     origin/votes_and_version

Danach ändern wir die Url des aktuellen origins auf unseren Fork.

.. code-block:: sh

   git remote origin git@github.com:syseleven/puppet-corosync.git

Jetzt haben wir ein Repo auf das wir Schreibzugriff haben in unserem Space.


Alle Änderungen, die wir nun pushen, landen in unserem Repo. Doch wozu dient upstreamn? Diesen benötigen wir, um mit dem "echten" Repository schritt zu halten und uns upstream-Änderungen zu ziehen. Dies ist bei kleinen/schnellen Änderungen sicherlich nicht wirklich nötig, aber man denke an die Linux-Kernel-Entwicklung. Man möchte vielleicht alle anderen Subsysteme, an denen man gerade nicht arbeitet aktuell halten.


Arbeitet man mit upstream-Remotes wird git fetch wichtiger. Oft möchte man erst einmal nur das upstream holen, aber noch nicht in den aktuellen Branch integrieren.

.. code-block:: sh

   git fetch upstream
   From https://github.com/puppet-community/puppet-corosync
    * [new branch]      master     -> upstream/master
   git branch -rl
     origin/HEAD -> origin/master
     origin/expected_votes
     origin/implement-version
     origin/master
     origin/votes_and_version
     upstream/master   ### <<<===

Z.B. vor einem Rebase mal ein diff machen um ein Gefühl für die Änderungen zu bekommen.

.. code-block:: sh

   git diff upstream/master..origin/votes_and_version
   ...

Am Ende der eigenen Entwicklung steht das Zurückliefern der Entwicklung. Die Art der Rückgabe hängt vom Projekt ab. Bei puppet-corosync genügt ein pull-Request, den man über github anstößt. Siehe [9]. Andere Projekte wie z.B. OpenStack [12], OpenContrail, Android oder Go basieren auf Gerrit [10] und git-review [11].

Git via Mail?
-------------

Dieses Freature kommt vor allem aus der Linux-Kernel-Entwicklung.

Patches zum Versenden via Mail erzeugen:

.. code-block:: sh

   git format-patch 72f90f1..HEAD
    0001-VideoPlayerActivity-remove-useless-statement.patch
    0002-don-t-sort-by-album-tracks-by-name-keep-files-order.patch
    0003-Thumbnailer-fix-typo.patch
    0004-MediaLibrary-add-an-extra-check-to-avoid-a-crash.patch
   
   cat 0001-VideoPlayerActivity-remove-useless-statement.patch
    From cbf7153d7519c48995bba9bc72d3104837ec070d Mon Sep 17 00:00:00 2001
    From: Edward Wang <edward.c.wang@compdigitec.com>
    Date: Sat, 10 Nov 2012 00:27:17 -0500
    Subject: [PATCH 1/4] VideoPlayerActivity: remove useless statement
    
    ---
     vlc-android/src/org/videolan/vlc/gui/video/VideoPlayerActivity.java | 1 -
     1 file changed, 1 deletion(-)
  
    diff --git a/vlc-android/src/org/videolan/vlc/gui/video/VideoPlayerActivity.java b/vlc-android/src/org/videolan/vlc/gui/video/VideoPlayerActivity.java
    index b35a1b8..760185a 100644
    --- a/vlc-android/src/org/videolan/vlc/gui/video/VideoPlayerActivity.java
    +++ b/vlc-android/src/org/videolan/vlc/gui/video/VideoPlayerActivity.java
    @@ -265,7 +265,6 @@ public class VideoPlayerActivity extends Activity {
                 LibVLC.useIOMX(this);
                 mLibVLC = LibVLC.getInstance();
             } catch (LibVlcException e) {
    -            e.printStackTrace();
                 Log.d(TAG, "LibVLC initialisation failed");
                 return;
             }
    --
    2.4.4


Kann eine Serie von Patches aus einer Mailbox anwenden:

.. code-block:: sh

   git am MAILBOX_FILE

Wendet einzelne Patches an:

.. code-block:: sh

   git apply PATCH_FILE

Mehr Arbeiten mit git
---------------------

Generell läuft es meistens auf das Arbeiten mit Feature-Branches hinaus. Dies ist selbst möglich wenn man git an SVN anbindet (ja das geht wirklich ;)).

Die Grundlagen bleiben wie gehabt. Nur in der echten Arbeit braucht man doch manchmal mehr.

Branches löschen
````````````````

Branch löschen:

.. code-block:: sh

   git branch -d BRANCHNAME

Der Versuch einen Branch zu löschen, der noch nicht gemerged wurde schlägt fehl.

Dies kann forciert werden:

.. code-block:: sh

   git branch -D BRANCHNAME

Beide Aktionen betreffen nur den lokalen Branch.

So löscht man einen Rremote-Branch. (Führender Doppelpunkt)

.. code-block:: sh

   git push origin :BRANCHNAME
                   ^

Arbeit zurücksetzen
```````````````````

Wie auch bei Rebase gilt hier: Kein Reset von bereits geteilten Commits.


Ab und an kommt es vor, dass man seine Arbeit zurücksetzen möchte. Z.B. wenn man gerade versehentlich ein paar Commits in den Master gemacht hat. Eigentlich wollte man seine Arbeit aber in einem Feature-Branch durchführen. Wir befunden uns also in master.

.. code-block:: sh

   git branch feature1
   git reset --hard HEAD~1
   git checkout feature1

Zuerst wird der Branch feature1 erzeugt. Anschließend master auf HEAD - 1 Commit zurückgesetzt. Danach in den Branch feature1 gewechselt.
Hier verwenden wir für den Ziel-Commit eine andere Schreibweise als gehabt. Statt eine fixe Commit-ID anzugeben, wird hier die Anzahl der Commits von aktuellen HEAD angegeben.
HEAD~1, HEAD~2, ... HEAD~N. Alternativ kann für HEAD~1 auch HEAD^ verwendet werden.


Sind die letzten 3 Commits quatsch, und man möchte sie einfach loswerden genügt ein

.. code-block:: sh

   git reset --hard HEAD~3
   git status
    nothing to commit, working directory clean

Wobei reset --hard immer den HEAD und die lokale Kopie verändert. Deine Änderungen sind also damit weg.

Möchte man nur die letzten 3 Commits rückgängig machen, die Arbeit aber behalten um daran weitere Änderungen durchzuführen und einen neuen Commit zu erstellen.

.. code-block:: sh

   git reset --soft HEAD~3
   git status
    YOUR CHANGES
   git commit -a

Reset hilft auch dabei einen Merge oder Pull rückgängig zu machen. Vor dem Merge wird der HEAD immer nach ORIG_HEAD gespeichert. Somit kann einfach dorthin zurück gesprungen werden, unabhängig davon wie viele Commits dazwischen liegen.

.. code-block:: sh

   git reset --hard ORIG_HEAD


Es ist sogar möglich einen Merge zurückzunehmen, den man in eine "unsaubere" Arbeitskopie übernommen hat. Unsauber im Sinne, es gibt noch uncommittete Änderungen. Macht man in einer solchen Arbeitskopie ein git pull, sind die eigenen Änderungen immer noch nicht committed, Upstream-Änderungen aber gemerged. Würde man wie oben reset --hard machen, würde man die aktuelle Arbeit auch verlieren. In einem solchen fall hilft

.. code-block:: sh

 git reset --merge ORIG_HEAD

Arbeit temporär einlagern
`````````````````````````

Möchte man in einer unsauberen Arbeitskopie die aktuellen Änderungen noch einmal zurücknehmen, also zum HEAD des Branches zurückspringen, die bisherige Arbeit aber nicht verlieren.

.. code-block:: sh
   
   git status
    YOUR CHANGES
   git stash save
    Saved working directory and index state WIP on master: 84ba7ee Blah
    HEAD is now at 84ba7ee Blah
   git stash list
    stash@{0}: WIP on master: 84ba7ee fdsag
   git status
    On branch master
    nothing to commit, working directory clean
   git stash pop
    On branch master
    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)
  
            modified:   file1
  
    no changes added to commit (use "git add" and/or "git commit -a")
    Dropped refs/stash@{0} (c21fa2cba0b71851de6fab909967b3a04512a551)
   git status
    YOUR CHANGES

In den Manpages stöbern
```````````````````````

Ich bin hier nur auf ein kleines Subset der Möglichkeiten von und mit git eingegangen. Es lohnt sich immer die manpages zu lesen und im Internet zu stöbern.

Links
-----

[1] https://www.atlassian.com/git/tutorials/comparing-workflows/

[2] http://endoflineblog.com/gitflow-considered-harmful

[3] https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

[4] https://www.atlassian.com/git/tutorials/merging-vs-rebasing/

[5] https://git-scm.com/book/de/v1/Git-Branching-Rebasing

[6] https://git-scm.com/book/de/v1

[7] https://git-scm.com/book/de/v1/Distribuierte-Arbeit-mit-Git-xxx-Distribuierte-Workflows

[8] http://nvie.com/posts/a-successful-git-branching-model/

[9] https://github.com/puppet-community/puppet-corosync/pull/141

[10] https://www.gerritcodereview.com/

[11] https://pypi.python.org/pypi/git-review

[12] http://docs.openstack.org/infra/manual/developers.html

[13] http://www.pro-linux.de/artikel/1/68/git-tutorium.html

[14] https://github.com/nojhan/liquidprompt
