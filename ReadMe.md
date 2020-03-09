# Amazon Simple Storage Service (S3)

## Cloud Storage

Beim Cloud Computing ist Cloud Storage ein Serviceangebot, mit dem ein Verbraucher Daten in Cloud-basierte Systeme, die von einem Service Provider verwaltet werden, lesen und schreiben kann. Cloud-Speicher ist normalerweise über einen Webdienst oder eine API zugänglich. Die zugrundeliegende Infrastruktur ist in der Regel ein virtualisierter Infrastruktur Stack mit Festplattenlaufwerken, die von einer Software-Serviceschicht verwaltet werden.

### Cloud Storage Typen

#### Object Storage

Bei der Objektspeicherung werden die Daten als einzelne Objekte und nicht als Dateihierarchie verwaltet (wie bei einem traditionellen Dateisystem). Jedes Objekt enthält die Daten selbst, Metadaten (beschreibende Daten) und einen global eindeutigen Bezeichner.

Aufgrund seiner flachen Dateistruktur ist die Objektspeicherung praktisch unbegrenzt skalierbar und ermöglicht die Speicherung großer Mengen unstrukturierter Daten. Die Daten werden oft über mehrere physische Systeme und Einrichtungen repliziert, was eine hohe Verfügbarkeit und Haltbarkeit gewährleistet.

![Object Storage Type](https://github.com/DanielW1987/aws-notes/blob/master/figures/object-storage-type.png)

Der Zugriff auf die Objektspeicherung erfolgt in der Regel über REST via HTTP. Objekte in S3 werden in einer flachen Struktur ohne Hierarchie gespeichert. Die Container der obersten Ebene, in denen Objekte gespeichert werden, werden als Buckets bezeichnet. Obwohl es keine Hierarchie gibt, unterstützt S3 das Konzept von Ordnern zur Organisation (Gruppierung von Objekten).

#### File Storage

Dateispeicherserver speichern Daten in einer hierarchischen Struktur mit Dateien und Ordnern. Auf die Daten wird zugegriffen als Datei-IDs über ein Netzwerk, wobei entweder der Server Message Block (SMB) für Windows oder das Network File System (NFS) für Unix/Linux verwendet wird.

Ein Dateisystem wird über das Netzwerk auf einen Client-Computer gemountet, wo es dann für das Lesen und Schreiben von Daten zugänglich wird. Dateien und Ordner können erstellt, aktualisiert und gelöscht werden.

![File Storage Type](https://github.com/DanielW1987/aws-notes/blob/master/figures/file-storage-type.png)

Auf einem gemounteten Dateisystem können nur Operationen auf Dateiebene durchgeführt werden, es ist nicht möglich, Befehle auf Blockebene auszugeben oder die zugrunde liegenden Speicher-Volumes zu formatieren oder zu partitionieren. Die Dateispeicherung ist einfach zu implementieren und zu verwenden und im Allgemeinen recht kostengünstig. Anwendungsfälle sind Web-Serving und Content-Management, gemeinsam genutzte Unternehmensverzeichnisse, Home-Laufwerke, Datenbank-Backups und große Datenanalyse-Workloads.

Das Amazon Elastic File System (EFS) ist ein einfacher, skalierbarer, elastischer Dateispeicher in der AWS-Cloud, der auf NFS basiert. EFS bietet die Möglichkeit, ein Dateisystem an viele EC2-Instanzen gleichzeitig zu mounten und kann einen hohen Gesamtdurchsatz und IOPS erreichen.

EFS ist ein regionaler AWS-Dienst und bietet eine hohe Verfügbarkeit und Haltbarkeit durch die redundante Speicherung von Daten in verschiedenen Verfügbarkeitszonen (AZs).

#### Block Storage

Die Daten werden in Blöcken innerhalb von Sektoren und Spuren gespeichert und verwaltet sowie von einem serverbasierten Betriebssystem gesteuert. Blockspeicher-Volumes erscheinen für das Betriebssystem als lokale Platten und können partitioniert und formatiert werden.

![Block Storage Type](https://github.com/DanielW1987/aws-notes/blob/master/figures/block-storage-type.png)

Blockspeicher wird normalerweise in SAN-Umgebungen (Storage Area Network) verwendet, die das Fibre-Channel-Protokoll (FC) sowie Ethernet-Netzwerke mit Protokollen wie iSCSI oder Fibre Channel over Ethernet (FCoE) verwenden. Blockspeicher ist in der Regel teurer als Objekt- oder Dateispeicher, bietet jedoch eine geringe Latenzzeit sowie eine hohe und konsistente Leistung. Die Kosten sind bei SAN-Implementierungen aufgrund der erforderlichen Spezialausrüstung oft am höchsten. Amazon Elastic Block Store (EBS) ist der AWS-Dienst für die Blockspeicherung. EBS bietet persistente Blockspeicher-Volumen für die Verwendung mit EC2-Instanzen.

Häufige Anwendungsfälle für die Blockspeicherung sind strukturierte Informationen wie Dateisysteme, Datenbanken, Transaktionsprotokolle, SQL-Datenbanken und VMs.

## S3 Intro

S3 ist ein AWS-globaler Objektspeicher zum Speichern und Abrufen von großer Datenmengen aus beliebigen Quellen.

### Use Cases

- Backup und Recovery
- Datenarchivierung
  - es existieren unterschiedliche Storage classes für schnellen Zugriff auf archivierte Daten
  - Lifecycle Policies für den Übergang von S3 zu Amazon Glacier
- Data Lake für Big Data Analytics
- Cloud-native Application Data (Web Content, Mobile App Content)

### S3 Disaster Recovery

Amazon S3 bietet eine robuste, sichere, globale Infrastruktur und eine robuste Disaster-Recovery-Lösung, die für einen überragenden Datenschutz entwickelt wurde. Cross-Region Replication (CRR) repliziert automatisch jedes S3-Objekt in ein Bucket, der sich in einer anderen AWS-Region befindet.

### S3 Components

- Bucket:
  - Container für Datenspeicherung
  - Region-spezifische Ressource
  - sollte idealerweise in der Region angelegt werden, wo es verwendet wird (gerine Latenz)
  - Ein Bucket kann undendlich viele Objekte beinhalten
  - Muss einen global eindeutigen Namen haben
  - Es gibt kein Konzept von Ordnern, die UI erlaubt uns virtuelle Ordner zur Strukturierung
- Object:
  - Jedes Objekt befindet sich in einem Bucket
  - Jedes Objekt besteht aus:
    - Key (Pfad des Objekts, beinhaltet also Bucket-Name, ggf. Ordner sowie Name des Objekts)
    - den zu speichernden Daten
    - Version ID
    - Metadaten (Name-Werte-Paare, die die Daten beschreiben)
  - Jedes Objekt kann bis zu 5 TB groß sein (für größere Dateien muss multipart Upload verwendet werden)

### S3 Speicherklassen (TODO)

Es gibt mehrere S3-Speicherklassen mit unterschiedlichen Niveaus der Verfügbarkeit, Haltbarkeit und Funktionen.

- S3 standard
- S3-IA
- S3 One Zone – IA
- Glacier

### S3 Security

#### Encryption at rest

Es gibt 4 Möglichkeiten, seine Objekte in S3 zu verschlüsseln:

- SSE-S3: Verschlüsselung mit Keys, die von AWS S3 erstellt und verwaltet werden (AES-256)
- SSE-KMS: Verschlüsselung mit Keys, die mit KMS erstellt und verwaltet werden
- SSE-C: Verschlüsselung mit selbst erstellten und verwalteten Keys (erfordert HTTPS)
- Client-side Encryption

Bei den Varianten SSE-S3, SSE-KMS und SSE-C werden die Objekte serverseitig verschlüsselt. Aus diesem Grund muss bei SSE-C der Schlüssel auch via HTTPS mitgesendet werden. Bei der Client-side Encryption findet die Verschlüsselung und Entschlüsselung vollständig client-seitig statt.

#### Encryption in transit

S3 bietet HTTP (nicht verschlüsselt) und HTTPS-Endpunkte zur Übertragung und Empfang von Objekten.

#### Policies

Bucket Policies (Resource base) und User oder IAM Policies (User based) sind zwei der Optionen für die Zugriffsbeschränkung, die für S3-Ressourcen zur Verfügung stehen. Beide verwenden eine JSON-basierte Zugriffsbeschränkungssprache.

Die JSON-basierten Bucket Policies beinhalten folgende Felder:

- Resources: Buckets und Objekte
- Actions: Satz an Aktionen, die erlaubt oder verboten werden sollen
- Effect: Allow oder Deny
- Principal: Konto oder Benutzer, den die Policy zugeordnet werden soll

Das folgende Beispiel zeigt eine Bucket Policy, die jedem unbekannten Benutzer Leseberechtigungen erteilt:

```json
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"PublicRead",
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::examplebucket/*"]
    }
  ]
}
```

Zur Generierung solcher JSON-basierter Plocies kann <https://awspolicygen.s3.amazonaws.com/policygen.html> verwendet werden.

### S3 Consistency Model

#### Read after write consistency for PUTs of new objects

Sobald ein Objekt via PUT geschrieben wurde, kann es gelesen werden (PUT 200 => GET 200).
Es kann eine Ausnahme geben, wenn vor dem Schreiben bereits via GET auf das Objekt zugegriffen wurde (GET 404 => PUT 200 => GET 404) - Eventually Consistency. Dieser Effekt kann bspw. durch Caching auftreten.

#### Eventual consistency for DELETEs and PUTs of existing objects

Wenn wir ein Objekt lesen, nachdem wir es mit PUT geändert haben, erhalten wir mglw. die alte Version (GET 200 (v1) => PUT 200 (v2) => GET 200 (v1)).

Wenn wir versuchen ein Objekt zu lesen, nachdem es mit DELETE gelöscht wurde, kann es sein, dass wir das Objekt immer noch lesen können (DELETE 200 => GET 200).

### Performance Tips

- Historisch: Jedes Objekt befindet sich in einer S3-Partition und für die beste Performance wollen wir die höchste Partitionsverteilung. Es ist daher empfohlen, den Namen eines Objekts mit einer randomisierten Zeichenkette beginnen zu lassen. Datum als Prefix ist nicht empfohlen.
- Aktuell: 3500 TPS for PUT and 5500 TPS for GET (TPS = Transactions per second). Es sind keine randomisierten Präfixe mehr notwendig.
- Multipart Upload für Objekte größer 100 MB.
- Benutzung von CloudFront als Cache für S3 Objekte
- Limitierung der S3 Performance durch KMS möglich (bei Verwendung von SSE-KMS)

### Kosten (TODO)

- Storage
- Requests
- Storage management pricing
- Data transfer pricing
- Transfer acceleration


## Amazon Glacier

- sehr kostengünstiger Storage Service
- kosteneffektiver als S3 (1/5 von S3)
- Daten werden in Archiven gespeichert und in einem Vault abgelget
- Ein Archiv ist jede Form von Daten (Foto, Video, Dokument, Datei etc.)
- Ein Archiv kann bis zu 40 TB groß sein
- In Glacier kann eine unbegrenzte Anzahl an Archiven gespeichert werden
- Downside: Es dauert sehr lange, die Daten herunterzuladen
- Daher: Eignet sich eher für die dauerhafte Abblage als für den häufigen Zugriff
- Upload und Download von Archiven in das Vault ist nur via API möglich
