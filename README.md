# MongoDB
> Verwendung der Datenbank MongoDB um die **json** Statistikdaten von srt-live-transmit direkt abspeichern zu können.  

## srt-live-transmit
`srt-live-transmit udp://224.0.0.1:9999?mode=listener srt://217.160.70.147:1995 -s 1000 -pf json -statsout:stats.log`  
So werden die Statistikdaten in der Datei stats.log laufend gespeichert. Ich suche noch nach einem Weg, diese Daten direkt in MongoDB zu speichern.  
`srt-live-transmit -q udp://224.0.0.1:9999 srt://217.160.70.147:1995 -s 500 -pf json`  
Der Parameter `-q  quiet mode` verhindert jede andere Ausgabe in der Konsole. Es werden nur die Statistikdaten geschrieben.

## Beispiel für einen Datensatz
```
{
  "sid": 118877286,
  "timepoint": "2021-05-20T17:38:27.777273+0000",
  "time": "6613604",
  "window": {
    "flow": "8183",
    "congestion": "8192",
    "flight": "2"
  },
  "link": {
    "rtt": "38.286",
    "bandwidth": "0.54",
    "maxBandwidth": "1000"
  },
  "send": {
    "packets": "994",
    "packetsUnique": "994",
    "packetsLost": "0",
    "packetsDropped": "0",
    "packetsRetransmitted": "0",
    "packetsFilterExtra": "0",
    "bytes": "1151996",
    "bytesUnique": "1151996",
    "bytesDropped": "0",
    "byteAvailBuf": "12280500",
    "msBuf": "32",
    "mbitRate": "0.689269",
    "sendPeriod": "8"
  },
  "recv": {
    "packets": "0",
    "packetsUnique": "0",
    "packetsLost": "0",
    "packetsDropped": "0",
    "packetsRetransmitted": "0",
    "packetsBelated": "0",
    "packetsFilterExtra": "0",
    "packetsFilterSupply": "0",
    "packetsFilterLoss": "0",
    "bytes": "0",
    "bytesUnique": "0",
    "bytesLost": "0",
    "bytesDropped": "0",
    "byteAvailBuf": "12286500",
    "msBuf": "0",
    "mbitRate": "0",
    "msTsbPdDelay": "120"
  }
}
```

## MongoDB installieren (Ubuntu 20.04)  
`sudo apt install mongodb`  

**Nach der Installation läuft die Datenbank sofort. (Ohne Passwort!) Für die StreamBox ist das o.k.**
`sudo systemctl status mongodb`  

## Linux Bash:  
`mongo --eval "printjson(db.serverStatus())" `  
MongoDB starten und zur mongo-Shell wechseln:  
`mongo` 
`> show dbs`  
`> use <database>`  
`> db.adminCommand('listDatabases')`  
`> show collections` entspricht `> show tables`  
`> db.getCollectionNames()`  
`> db.<collection>.find()` 

### Daten von srt-live-transmit in Datenbank einfügen
#### direkt von der Linux Bash Shell
`mongo srt_db --eval 'db.transmit.insert(
{"sid":749640009,"timepoint":"2021-05-17T12:56:23.135103+0200","time":"336106","window":{"flow":"7637","congestion":"8192","flight":"6"},"link":{"rtt":"35.4","bandwidth":"1.164","maxBandwidth":"1000"},"send":{"packets":"502","packetsUnique":"502","packetsLost":"0","packetsDropped":"0","packetsRetransmitted":"0","packetsFilterExtra":"0","bytes":"542848","bytesUnique":"542848","bytesDropped":"0","byteAvailBuf":"12280500","msBuf":"17","mbitRate":"0.933062","sendPeriod":"7"},"recv":{"packets":"0","packetsUnique":"0","packetsLost":"0","packetsDropped":"0","packetsRetransmitted":"0","packetsBelated":"0","packetsFilterExtra":"0","packetsFilterSupply":"0","packetsFilterLoss":"0","bytes":"0","bytesUnique":"0","bytesLost":"0","bytesDropped":"0","byteAvailBuf":"12286500","msBuf":"0","mbitRate":"0","msTsbPdDelay":"120"}} 
)'`  

Achtung: Beim Aufruf aus der Linux Bash-Shell muss der Ausdruck hinter `--eval` mit einfachen Hochkommas `'` eingeschlossen werden, doppelte Hochkommas `"` funktionieren nicht! 

#### per Skript
```
#!/bin/bash
# srt-live-transmit \
#   ffmpeg -i 'rtsp://192.168.95.55:554/1/h264major' -c copy -f mpegts 'srt://localhost:1995'
#
#   srt://:1995?mode=listener \
#   srt://<restreamServer>:<Port> \
#     -s 1000 -pf json -statsout:stats.log
#     Damit werden die Statistikdaten in die Datei stats.log geschrieben.
#     This will write the statistics data to the stats.log file.

# MongoDB
db='srt_db'
col='transmit'

while [ true ]; do
  # Den letzten Datensatz aus der Datei stats.log holen und der Variable last_stat zuweisen.
  # Get the last record from the stats.log file and assign it to the last_stat variable.
  last_stat=$( tail -n 1 stats.log )
  # prüfen, ob der Datensatz komplett ist (gleiche Anzahl '{' und '}' )
  if ! [ $(echo $last_stat | grep -o '{' | wc -w) == $(echo $last_stat | grep -o '}' | wc -w) ]
    then
      echo "json error"
    else
      # Zeitstempel auslesen
      timepointNew=$(jq '.timepoint' <<< $last_stat)
      # prüfen, ob der aus der Datei gelesene Datensatz einen neuen Zeitstempel hat
      if [ "$timepointNew" != "$timepointOld" ]; then
        # echo $timepoint
        # Ausgabe in Terminal zur Kontrolle
        echo $last_stat | jq '.'
        # Datensatz in die Datenbank srt_db in die Collection transmit schreiben
        command="db.${col}.insert(${last_stat})"
        # echo $command
        mongo $db --eval ${command}
        # Zeitstempel für den Vergleich aktualisieren
        timepointOld=$timepointNew
      fi
  fi
  sleep 1
done

# Beispiel für eine Abfrage aus der Datenbank
# Query the most recent record from MongoDB
# db.transmit.find().sort({timepoint:-1}).limit(1).pretty()
#
# Mit diesem Aufruf aus der Bash Shell bekomme ich sauberen json-Code zurück:
# mongo --quiet srt_db --eval '(db.transmit.find({},{_id:0}).sort({timepoint: -1}).limit(1))'
``` 

### Datenbankabfrage
direkt aus der Linux Shell:  
`mongo srt_db --eval 'db.transmit.find().sort({timepoint: -1}).limit(1)'`  
mit `--quiet` als erste Option, vor `--eval` wird der Header nicht mit ausgegeben.  
aus der Mongo-Shell:  
`> db.transmit.find().sort({timepoint:-1}).limit(1).pretty()`  
Damit sortiere ich die Datensätze in der Collection (bzw. Tabelle) "transmit" nach Zeitstempel abwärts, erhalte also den aktuellsten Datensatz, genau einen wegen limit(1). "pretty()" macht die Ausgabe schöner, wird aber nicht benötigt. Um die vier Headerzeilen noch zu entfernen, verwende ich weiter unten "--quiet".
```
mongo srt_db --eval 'db.transmit.find().sort({timepoint: -1}).limit(1).pretty()'
MongoDB shell version v3.6.8
connecting to: mongodb://127.0.0.1:27017/srt_db
Implicit session: session { "id" : UUID("a073a154-759b-47dd-9344-3dbf4d898a8e") }
MongoDB server version: 3.6.8
{
	"_id" : ObjectId("60a24e314da1fc66640661c8"),
	"sid" : 749640009,
	"timepoint" : "2021-05-17T12:56:46.564023+0200",
	"time" : "359535",
	"window" : {
		"flow" : "7637",
		"congestion" : "8192",
		"flight" : "0"
	},
	"link" : {
		"rtt" : "33.809",
		"bandwidth" : "1.128",
		"maxBandwidth" : "1000"
	},
	"send" : {
		"packets" : "501",
		"packetsUnique" : "501",
		"packetsLost" : "0",
		"packetsDropped" : "0",
		"packetsRetransmitted" : "0",
		"packetsFilterExtra" : "0",
		"bytes" : "547316",
		"bytesUnique" : "547316",
		"bytesDropped" : "0",
		"byteAvailBuf" : "12282000",
		"msBuf" : "17",
		"mbitRate" : "0.925541",
		"sendPeriod" : "7"
	},
	"recv" : {
		"packets" : "0",
		"packetsUnique" : "0",
		"packetsLost" : "0",
		"packetsDropped" : "0",
		"packetsRetransmitted" : "0",
		"packetsBelated" : "0",
		"packetsFilterExtra" : "0",
		"packetsFilterSupply" : "0",
		"packetsFilterLoss" : "0",
		"bytes" : "0",
		"bytesUnique" : "0",
		"bytesLost" : "0",
		"bytesDropped" : "0",
		"byteAvailBuf" : "12286500",
		"msBuf" : "0",
		"mbitRate" : "0",
		"msTsbPdDelay" : "120"
	}
}

```
**Ein Problem ist die von MongoDB übergebene "interne Datenbank-ID" mit den Klammern () "\_id". Damit kommt der Linux Shell json-Parser "jq" nicht zurecht. Deshalb sollte man mit "find({},{\_id:0})" zuerst diese ID wieder löschen.**    
Mit diesem Aufruf bekomme ich jetzt sauberen json-Code zurück:  
`mongo --quiet srt_db --eval '(db.transmit.find({},{_id:0}).sort({timepoint: -1}).limit(1))'`  
diesen Ausdruck dann noch durch eine Pipe zu jq:  
`| jq '.link'` zeigt mir z.B. die Linkdaten an.  
`{  
  "rtt": "33.809",  
  "bandwidth": "1.128",  
  "maxBandwidth": "1000"  
}`  
`| jq '.link.bandwidth' ` gibt mir ausschließlich die Bandbreite zurück:  
`"1.128"`  
Besser wäre es, alle Daten ordentlich in ein Array[] einzulesen um sie dann gezielt anzeigen zu können.  
Dies soll aber PHP übernehmen, da ich die Werte gern als Browser-Dock direkt in OBS anzeigen möchte.  
>verfügbare Bandbreite von der StreamBox zum RestreamServer (link.bandwidth)  
>Pingwerte (link.rtt)  
>genutzte Bandbreite (send.mbitRate)  
