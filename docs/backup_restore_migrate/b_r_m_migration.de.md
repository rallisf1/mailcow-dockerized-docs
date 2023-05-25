## ...innerhalb desselben Systemtyps (x86 auf x86, ARM64 auf ARM64 usw.)

!!! warning "Warnung"
    Diese Anleitung geht davon aus, dass Sie einen bestehenden mailcow-Server (Quelle) auf einen neuen, leeren Server (Ziel) migrieren wollen. Seien Sie sich bewusst, dass, wenn Sie dieser Anleitung folgen, **ALLE** Docker Volumes (standardmäßig `/var/lib/docker/volumes`) und somit alle Daten in diesen Volumes gelöscht werden.

!!! tip "Tipp"
    Alternativ können Sie das Skript `./helper-scripts/backup_and_restore.sh` verwenden, um ein vollständiges Backup auf der Quellmaschine zu erstellen, dann installieren Sie mailcow auf der Zielmaschine wie gewohnt, kopieren Sie Ihre `mailcow.conf` (aus dem Backup Ordner) und verwenden das gleiche Skript, um Ihr Backup auf der Zielmaschine wiederherzustellen.

**1\.**
Befolgen Sie die [Installationsanleitung](../getting-started/i_u_m_install.de.md) von Docker und Compose.

**2\.** Stoppen Sie Docker und stellen Sie sicher, dass Docker gestoppt wurde:
```
systemctl stop docker.service
systemctl status docker.service
```

**3\.** Führen Sie die folgenden Befehle auf dem Quellcomputer aus (achten Sie darauf, die Schrägstriche am Ende des ersten Pfadparameters wie unten gezeigt hinzuzufügen). 

!!! danger "Vorsicht"
    Dieser Befehl (der Zweite) löscht alles, was sich bereits unter `/var/lib/docker/volumes` auf dem Zielserver befindet. Stellen Sie also sicher, dass sich nichts Wichtiges mehr auf dem Zielserver befindet, bevor Sie die folgenden Befehle ausführen!

```
rsync -aHhP --numeric-ids --delete /opt/mailcow-dockerized/ root@target-machine.example.com:/opt/mailcow-dockerized
rsync -aHhP --numeric-ids --delete /var/lib/docker/volumes/ root@target-machine.example.com:/var/lib/docker/volumes
```

**4\.** Fahren Sie mailcow herunter und stoppen Sie Docker auf dem Quellrechner.
=== "docker compose (Plugin)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker compose down
    systemctl stop docker.service
    ```

=== "docker-compose (Standalone)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker-compose down
    systemctl stop docker.service
    ```

**5\.** Wiederholen Sie Schritt 3 mit den gleichen Befehlen. Dies geht wesentlich schneller als beim ersten Mal, da diesmal nur die Unterschiede (Diffs) synchronisiert werden müssen.

**6\.** Wechseln Sie auf den Zielrechner und starten Sie Docker.
```
systemctl start docker.service
```

**7\.** Laden Sie die Docker Images für mailcow auf dem Zielserver herunter.
=== "docker compose (Plugin)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker compose pull
    ```

=== "docker-compose (Standalone)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker-compose pull
    ```

**8\.** Starten Sie den gesamten mailcow-Stack und warten Sie einen Moment.
=== "docker compose (Plugin)"

    ``` bash
    docker compose up -d
    ```

=== "docker-compose (Standalone)"

    ``` bash
    docker compose up -d
    ```
Wenn alles funktioniert hat, sollte mailcow auf dem neuen Server genauso aussehen und funktionieren wie auf dem alten Server!

**9\.** Zum Schluss ändern Sie Ihre DNS-Einstellungen so, dass sie auf den Zielserver zeigen. Überprüfen und ändern Sie ggf. die Variable `SNAT_TO_SOURCE` in der Datei `mailcow.conf` im Verzeichnis mailcow-dockerized, da SOGo sonst nicht richtig funktioniert, wenn die ausgehende IP unterschiedlich ist.

**10\.** Freude an mailcow auf dem neuen Server haben! :grinning:

---

## ...auf einen anderen Systemtyp (x86 auf ARM64 und umgekehrt)

!!! warning "Warnung"
    Diese Anleitung geht davon aus, dass Sie einen bestehenden mailcow-Server (Quelle) auf einen neuen, leeren Server (Ziel) migrieren wollen. Seien Sie sich bewusst, dass, wenn Sie dieser Anleitung folgen, **ALLE** Docker Volumes (standardmäßig `/var/lib/docker/volumes`) und somit alle Daten in diesen Volumes gelöscht werden.

!!! info "Hinweis"
    Alternativ können Sie das Skript `./helper-scripts/backup_and_restore.sh` verwenden, um ein vollständiges Backup auf der Quellmaschine zu erstellen. 

    In diesem Fall sollten Sie jedoch Rspamd aus dem Backup-Prozess ausschließen (verwenden Sie nicht `all` als Backup-Option), da es sonst aufgrund von Architekturunterschieden zu Problemen während des Startvorgangs kommen kann.

    Installieren Sie dann mailcow auf der Zielmaschine wie gewohnt, kopieren Sie Ihre mailcow.conf (aus dem Backup-Ordner) und verwenden Sie dasselbe Skript, um Ihr Backup auf der Zielmaschine wiederherzustellen.

**1\.**
Befolgen Sie die [Installationsanleitung](../getting-started/i_u_m_install.de.md) von Docker und Compose.

**2\.** Stoppen Sie Docker und stellen Sie sicher, dass Docker gestoppt wurde:
```
systemctl stop docker.service
systemctl status docker.service
```

**3\.** Führen Sie die folgenden Befehle auf dem Quellcomputer aus (achten Sie darauf, die Schrägstriche am Ende des ersten Pfadparameters wie unten gezeigt hinzuzufügen). 

!!! danger "Vorsicht"
    Dieser Befehl (der Zweite) löscht alles, was sich bereits unter `/var/lib/docker/volumes` auf dem Zielserver befindet. Stellen Sie also sicher, dass sich nichts Wichtiges mehr auf dem Zielserver befindet, bevor Sie die folgenden Befehle ausführen!

```
rsync -aHhP --numeric-ids --delete /opt/mailcow-dockerized/ root@target-machine.example.com:/opt/mailcow-dockerized
rsync -aHhP --numeric-ids --exclude --exclude=*rspamd-vol* --delete /var/lib/docker/volumes/ root@target-machine.example.com:/var/lib/docker/volumes
```

!!! info "Hinweis"
    Im Vergleich zur Migration auf die gleiche Architektur wird hier das Rspamd-Volume ausgelassen, da es sonst zu Problemen beim Start des Rspamd-Containers kommen kann.

    Die entsprechenden Daten des Rspamd-Volumes werden jedoch beim ersten Start auf der neuen Architektur automatisch regeneriert, so dass es keinen Unterschied in der Nutzung geben sollte.


**4\.** Fahren Sie mailcow herunter und stoppen Sie Docker auf dem Quellrechner.
=== "docker compose (Plugin)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker compose down
    systemctl stop docker.service
    ```

=== "docker-compose (Standalone)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker-compose down
    systemctl stop docker.service
    ```

**5\.** Wiederholen Sie Schritt 3 mit den gleichen Befehlen. Dies geht wesentlich schneller als beim ersten Mal, da diesmal nur die Unterschiede (Diffs) synchronisiert werden müssen.

**6\.** Wechseln Sie auf den Zielrechner und starten Sie Docker.
```
systemctl start docker.service
```

**7\.** Laden Sie die Docker Images für mailcow auf dem Zielserver herunter.
=== "docker compose (Plugin)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker compose pull
    ```

=== "docker-compose (Standalone)"

    ``` bash
    cd /opt/mailcow-dockerized
    docker-compose pull
    ```

**8\.** Starten Sie den gesamten mailcow-Stack und warten Sie einen Moment.
=== "docker compose (Plugin)"

    ``` bash
    docker compose up -d
    ```

=== "docker-compose (Standalone)"

    ``` bash
    docker compose up -d
    ```
Wenn alles funktioniert hat, sollte mailcow auf dem neuen Server genauso aussehen und funktionieren wie auf dem alten Server!

**9\.** Zum Schluss ändern Sie Ihre DNS-Einstellungen so, dass sie auf den Zielserver zeigen. Überprüfen und ändern Sie ggf. die Variable `SNAT_TO_SOURCE` in der Datei `mailcow.conf` im Verzeichnis mailcow-dockerized, da SOGo sonst nicht richtig funktioniert, wenn die ausgehende IP unterschiedlich ist.

**10\.** Freude an mailcow auf dem neuen Server haben! :grinning:
