# OpenStack Curl Command Examples

```bash
# create volume
curl -g -i \
  -X POST http://192.168.77.63/volume/v2/2a97f77210a54665ba9df8a505df6624/volumes \
  -H "User-Agent: python-cinderclient" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "X-Auth-Token: {SHA1}5b729489a9259aae07eb591361c474bdad29e71d" \
  -d '
    {"volume":
      {"status": "creating",
       "user_id": null,
       "name": "v-m-3",
       "imageRef": null,
       "availability_zone": null,
       "description": null,
       "multiattach": false,
       "attach_status": "detached",
       "volume_type": null,
       "metadata": {},
       "consistencygroup_id": null,
       "source_volid": null,
       "snapshot_id": null,
       "project_id": null,
       "source_replica": null,
       "size": 1
      }
    }'
```

```bash
# force detach
curl -g -i \
  -X POST \
  http://192.168.77.63/volume/v2/2a97f77210a54665ba9df8a505df6624/volumes/c8fd9bf0-12d9-4152-b2bd-dec2503e21c4/action \
  -H "User-Agent: python-cinderclient" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "X-Auth-Token: gAAAAABaXsh6TRclvEnyehxFQvmYejDIbTU0V8NilpRpDZj52Qb7SD5Dlvmieh6eL45bueXeu23qUCcDhghJgZKkC4gVgfJC45pHJLnUrYN7qkpwcKyyLYcqvXYvwXLGX-Mb0BNSDBpeMTf96bYTEBPIcmknez4xykpvXJRORI7qDW_d0x953Hk" \
  -d '
    {"os-force_detach":
      {"attachment_id":
        "0d207301-4fce-4da0-b4db-ea848d3cd6c7"
      }
    }'
```

```bash
# get token
curl -i \
  -H "Content-Type: application/json" \
  -d '
    { "auth": {
        "identity": {
          "methods": ["password"],
          "password": {
            "user": {
              "name": "admin",
              "domain": { "id": "default" },
              "password": "welcome"
            }
          }
        },
        "scope": {
          "project": {
            "name": "admin",
            "domain": { "id": "default" }
          }
        }
      }
    }' \
  "http://192.168.77.63/identity/v3/auth/tokens"
```

