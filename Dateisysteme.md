# Dateisysteme



#### Unix

Persistenter Speicher unter Unix: Daten werden in sogennanten **Inodes** gespeichert.

Ein Inode ist eine grundlegende Datenstruktur zur Verwaltung von Dateisystemen mit Unixartigen Betriebssystemen. 



Jeder Inode wird innerhalb eines **Dateisystems** eindeutig durch seine *Inode-Nummer* identifiziert und jeder Namenseintrag in einem Verzeichnis verweist auf genau einen Inode. Dieser enthält Metadaten (wie die Berechtigungen) einer Datei und verweist auf die Daten der Datei beziehungsweise die Dateiliste des Verzeichnisses.



**Warum Inodes?** Logischerweise kann man nicht alle Dateien hintereinanderspeichern.

| File 1 | File2 | File3 | ... | File10 | -empty- |
| ------ | ----- | ----- | --- | ------ | ------- |

Wenn sich eine Datei vergrößert, müsste man den gesamten Diskspeicher  verschieben!

| File1 | File1 | File2 | File3 | ... | File10 |
| ----- | ----- | ----- | ----- | --- | ------ |

Man hängt das Stück der Datei einfach hinten heran. Dann muss man nämlich nichts verschieben.

| File1-1 | File2 | File3 | ... | File10 | File1-2 |
| ------- | ----- | ----- | --- | ------ | ------- |

Die gesamte Festplatte ist in gleichgroße Blöcke aufgeteilt. Wenn eine Datei größer ist als ein Block, dann wird sie in mehreren Blöcken gespeichert. **Jede Datei hat einen ihr zugeordneten Inode, in dem gespeichert ist, aus welchen Blöcken die Datei besteht.** 



Alle Inodes sind gleich groß und sind im sogenannten **Super Block** gespeichert. Die einheitliche Größe macht es einem einfach, die Inodes später wiederzufinden.



Inodes speichern mehr als nur Daten, zum Beispiel speichern sie

- wann eine Datei erstellt wurde,

- wie groß sie ist,

- wie viele Links es zu dieser Datei gibt oder

- wer die Datei erstellt hat.



Also hat jeder Inode Blocks und Metadaten.

Wenn man die Daten der Datei möchte kann der Inode einem sagen in welchem Block sie gespeichert ist/sind. Oder wenn man mehr über die Datei wissen möchte gibt er einem die Metadaten.



Dateiberechtigungen sind in Unix in 2 Kategorien zu unterscheiden:

- Flüchtiger Speicher

- Persistenter Speicher

Bei flüchtigem Speicher ist es einfach: Diese Berechtigungen nimmt sich das Betriebssystem aus dem persistenten Speicher und sind im Kernel-Space gemanaged.



Wie vorher erwähnt hat jeder Inode dieselbe Größe. Jeder Inode hat ein Array aus 12 Block Pointers. Die Standardgröße für so einen Block pointer ist 4kb. Also hat ein Inode

4 * 12 = 48kb. Doch was, wenn eine Datei größer ist als 12 Blocks?



Jeder Inode hat zusätzlich zu den 12 Block pointers auch noch 3 sog. **Indirect Block Pointers**. Jeder dieser Indirect Block Pointers zeigt auf einen Block voller Block Pointers, welches die Kapazitäten eines Inodes sehr stark vergrößtert. Indirekte Blöcke können auch auf andere Indirekte Blöcke zeigen, und diese wieder auf andere... je nach verstrickungsgeneration nennt man das dann Double, Triple oder Quad Indirect Block. Dieses Verfahren gibt einem die Möglichkeit Dateien zu erstellen die bis zu 4TB groß sind.



Nebeneffekte von diesem Inode-Verfahren gibt es auch. Zu einem kann der Speicher der Festplatte knapp werden, ohne dass die Disk wirklich voller Daten ist: Der Super Block Table hat eine limitierte Größe, und wenn man viele sehr kleine Dateien hat kann es sein dass der Table voll ist. Die Platte ist nicht mehr beschreibbar obwohl kaum Daten drauf sind. 



Inodes sind allerdings nur in Unix vertreten. Apple und Microsoft haben ihre eigenen Dateisysteme.



Apple benutzt HFS+, welches eine **Catalog File**, welches dem Inode-Prinzip ähnelt.

Microsoft benutzt NTFS mit sog. File Record Attributes, welche den selben Zweck erfüllen.



Außerdem sind Links auch im Inode verankert. 

Möchte ich ``sh`` ausführen,  führe ich

```/bin/sh
/bin/sh
```

aus. Allerdings ist `/bin` ein Link zu `/usr/bin`

Hier der Beweis:

```
$ ls -la /bin
lrwxrwxrwx 1 root root 7 Aug 19  2021 /bin -> usr/bin
```

`/bin` ist ein **Link** zum Inode `/usr/bin`.



**Merke:**

Inodes enthalten Informationen darüber, aus welchen Blöcken eine Datei besteht und können ausdrücken, welche Eigenschaften die Datei hat, z.B. Größe, rwx, Links. Alle Inodes sind im Super Block Table gespeichert. Wenn eine Datei zu groß für einen Inode ist, benutzt der Inode Blöcke als zusätzlichen Speicher. Momentan kann man mit dem Unix File System bis zu 4 TB große Dateien besitzen. Wenn man sehr viele kleine Dateien hat, kann der Super Block volllaufen und die Festplatte ist nicht weiter beschreibbar.



#### Dos

**NTFS** (New Technology File System) ist ein proprietäres, von Microsoft entwickeltes Dateisystem. 



Die Features dieses Dateisystems sind

- **Skalierbar** mit theoretischen Dateigrößen bis zu 16 ExaByte/1.000.000 Terabyte (praktisch allerdings "nur" 256TB möglich)

- **Journaling**: ( $LogFile ) ist eine Datei die kein Nutzer sehen kann, allerdings auf der Festplatte gespeichert ist: Darin stehen die Operationen, die auf der Platte getätigt wurden(copy,delete,create usw.). Sinn dahinter: Wenn das System ausfällt passiert wenig bis kein Datenverlust. 



Quellen:

https://stackoverflow.com/questions/14151584/where-does-unix-store-open-file-offsets-permissions-and-list-of-names-links-

https://de.wikipedia.org/wiki/Inode
