# Vault HA Cluster with Integrated Storage (Raft)

**[TECH PREVIEW]** Currently, Vault's Integrated Storage is a _Technical Preview (BETA)_ feature. To explore this new persistent storage, [download Vault 1.2 or later](https://releases.hashicorp.com/vault/).

These assets are provided to perform the tasks described in the [Vault HA with Raft Storage Backend](https://learn.hashicorp.com/vault/operations/raft-storage) guide.

This guide provides two options to explore the raft storage backend:

- **Option 1:** Create a Vault HA cluster locally on your machine --> Use the scripts in the **`local-test`** folder.
- **Option 2:** Create a Vault HA cluster on AWS using Terraform --> Use the Terraform files in the **`terraform-aws`** folder.



## Taking a Vault data snapshot

To take a snapshot of the Vault data, use the `raft snapshot save` command: `vault operator raft snapshot save <snapshot_name>`

**Example:**

```plaintext
$ vault operator raft snapshot save 2019-JULY-04.snapshot
```

Now, delete the secrets at `kv/apikey`:

```sh
$ vault kv metadata delete kv/apikey

# Verify that the data no longer exists:
$ vault kv get kv/apikey
No value found at kv/data/apikey
```

## Restoring data from a snapshot

To recover the lost data from a snapshot, use the `raft snapshot restore`
command:

```plaintext
vault operator raft snapshot restore <snapshot_name>
```

**Example:**

```plaintext
$ vault operator raft snapshot restore 2019-JULY-04.snapshot
```

Now, you the data should be recoverable at `kv/apikey`:

```plaintext
$ vault kv get kv/apikey

====== Metadata ======
Key              Value
---              -----
created_time     2019-07-02T05:50:39.038931Z
deletion_time    n/a
destroyed        false
version          2

======= Data =======
Key           Value
---           -----
expiration    365 days
webapp        ABB39KKPTWOR832JGNLS02
```


## Remove a cluster member

To join the HA cluster, you executed the `raft join <leader_addr>` command. To remove a node from the cluster, execute the `raft remove-peer` command: `vault operator raft remove-peer <node_id>`


**Example:**

```plaintext
$ vault operator raft remove-peer node3
Peer removed successfully!
```

Now, `node3` is removed from the HA cluster.

```plaintext
$ vault operator raft configuration -format=json | jq
{
  ...
      "servers": [
        {
          "address": "127.0.0.2:8201",
          "leader": true,
          "node_id": "node2",
          "protocol_version": "3",
          "voter": true
        },
        {
          "address": "127.0.0.4:8201",
          "leader": false,
          "node_id": "node4",
          "protocol_version": "3",
          "voter": true
        }
      ]
    ...
}
```