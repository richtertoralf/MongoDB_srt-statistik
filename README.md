# MongoDB

installieren (Ubuntu 20.04)  
`sudo apt install mongodb`  

`sudo systemctl status mongodb`  

Linux Bash:  
`mongo --eval "printjson(db.serverStatus())" `  
MongoDB starten:  
`mongo` 
`> show dbs`  
`> use <database>`  
`> db.adminCommand('listDatabases')`  
`> show collections` oder `> show tables`  
`> db.getCollectionNames()`  
`> db.<collection>.find()` 



Daten in Datenbank einfügen:
`mongo srt_db --eval 'db.transmit.insert( {"sid":749640009,"timepoint":"2021-05-17T12:56:23.135103+0200","time":"336106","window":{"flow":"7637","congestion":"8192","flight":"6"},"link":{"rtt":"35.4","bandwidth":"1.164","maxBandwidth":"1000"},"send":{"packets":"502","packetsUnique":"502","packetsLost":"0","packetsDropped":"0","packetsRetransmitted":"0","packetsFilterExtra":"0","bytes":"542848","bytesUnique":"542848","bytesDropped":"0","byteAvailBuf":"12280500","msBuf":"17","mbitRate":"0.933062","sendPeriod":"7"},"recv":{"packets":"0","packetsUnique":"0","packetsLost":"0","packetsDropped":"0","packetsRetransmitted":"0","packetsBelated":"0","packetsFilterExtra":"0","packetsFilterSupply":"0","packetsFilterLoss":"0","bytes":"0","bytesUnique":"0","bytesLost":"0","bytesDropped":"0","byteAvailBuf":"12286500","msBuf":"0","mbitRate":"0","msTsbPdDelay":"120"}} )'`  

`> db.transmit.insert( {"sid":749640009,"timepoint":"2021-05-17T12:56:23.135103+0200","time":"336106","window":{"flow":"7637","congestion":"8192","flight":"6"},"link":{"rtt":"35.4","bandwidth":"1.164","maxBandwidth":"1000"},"send":{"packets":"502","packetsUnique":"502","packetsLost":"0","packetsDropped":"0","packetsRetransmitted":"0","packetsFilterExtra":"0","bytes":"542848","bytesUnique":"542848","bytesDropped":"0","byteAvailBuf":"12280500","msBuf":"17","mbitRate":"0.933062","sendPeriod":"7"},"recv":{"packets":"0","packetsUnique":"0","packetsLost":"0","packetsDropped":"0","packetsRetransmitted":"0","packetsBelated":"0","packetsFilterExtra":"0","packetsFilterSupply":"0","packetsFilterLoss":"0","bytes":"0","bytesUnique":"0","bytesLost":"0","bytesDropped":"0","byteAvailBuf":"12286500","msBuf":"0","mbitRate":"0","msTsbPdDelay":"120"}}  ) `  

Achtung: Beim Aufruf aus der Linux Bash-Shell muss der Ausdruck hinter `--eval` mit einfachen Hochkommas `'` eingeschlossen werden, `"` funktionieren nicht.

Datenbankabfrage
aus der Linux Shell:  
`mongo srt_db --eval 'db.transmit.find().sort({timepoint: -1}).limit(1)'`  
mit `--quiet` als erste Option, vor `--eval` wird der Header nicht mit ausgegeben.  
aus der Mongo-Shell:  
`> db.transmit.find().sort({timepoint:-1}).limit(1).pretty()`  

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
