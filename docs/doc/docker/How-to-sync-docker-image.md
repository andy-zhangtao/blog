# How to sync docker hub image easily ?

## Modify github repostry file via API

1. Get the last commit. e.g. invoke the api `https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/refs/heads/main` Use `GET` method.

   If invoke success, then will return a json object like below:

```json
{
  "ref": "refs/heads/main",
  "node_id": "MDM6UmVmMzI1MTU2OTM0OnJlZnMvaGVhZHMvbWFpbg==",
  "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/refs/heads/main",
  "object": {
    "sha": "970e75be2dada828114ac59d845a23cde29e90ca",
    "type": "commit",
    "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/commits/970e75be2dada828114ac59d845a23cde29e90ca"
  }
}
```

Then get `object` -> `sha`

2. Get repostry base tree of specify sha. The api is `https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/commits/970e75be2dada828114ac59d845a23cde29e90ca` Use `GET` method.

   The api will also return a json object. The `tree` -> `sha` is what we need.

```json
{
  "sha": "970e75be2dada828114ac59d845a23cde29e90ca",
  "node_id": "C_kwDOE2GARtoAKDk3MGU3NWJlMmRhZGE4MjgxMTRhYzU5ZDg0NWEyM2NkZTI5ZTkwY2E",
  "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/commits/970e75be2dada828114ac59d845a23cde29e90ca",
  "html_url": "https://github.com/andy-zhangtao/sync-docker-hub/commit/970e75be2dada828114ac59d845a23cde29e90ca",
  "author": {
    "name": "andy zhangtao",
    "email": "ztao8607@gmail.com",
    "date": "2022-10-27T14:50:57Z"
  },
  "committer": {
    "name": "andy zhangtao",
    "email": "ztao8607@gmail.com",
    "date": "2022-10-27T14:50:57Z"
  },
  "tree": {
    "sha": "fd5c681378c78bed29e0b2ed2bf6efbe063b301b",
    "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/trees/fd5c681378c78bed29e0b2ed2bf6efbe063b301b"
  },
  "message": "commit with rest api",
  "parents": [
    {
      "sha": "61a11f24143f66815f120c2764b334ad636fbfbf",
      "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/commits/61a11f24143f66815f120c2764b334ad636fbfbf",
      "html_url": "https://github.com/andy-zhangtao/sync-docker-hub/commit/61a11f24143f66815f120c2764b334ad636fbfbf"
    }
  ],
  "verification": {
    "verified": false,
    "reason": "unsigned",
    "signature": null,
    "payload": null
  }
}
```

3. Create a new tree use the last tree sha code. The api is `https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/trees` and use `POST` method. File content added in body.

```json
{
  "base_tree": "fd5c681378c78bed29e0b2ed2bf6efbe063b301b",
  "tree": [
    {
      "path": "Dockerfile.2",
      "mode": "100644",
      "type": "blob",
      "content": "FROM vikings/nginx-ingress-controller-amd64:0.30.0 \nENV  auth=ztao8607@gmail.com\nENV modify=1666879975902 \nUSER root\nRUN  apk add  zlib zlib-dev unzip git gcc musl-dev && \\n    git config --global http.sslVerify \"false\" && \\n    luarocks install lua-zlib\nRUN  sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories    \nUSER www-data"
    }
  ]
}
```

    Please note that:
    + `base_tree` is the sha which get in Step 2.
    + Path is the fully path.
    + Typicall the mode is 100644.
    + If upload a text-plaint file, then type is blob.
    + Content must be a singal line.

    If invoke success, then will return

```json
{
  "sha": "a3328d8bc90acbac72992b184ae9298fccbe9535",
  "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/trees/a3328d8bc90acbac72992b184ae9298fccbe9535",
  "tree": [
    {
      "path": ".travis.yml",
      "mode": "100644",
      "type": "blob",
      "sha": "3db26de5631fb26c3e8408f4d4ef5f1ed3d93385",
      "size": 1246,
      "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/blobs/3db26de5631fb26c3e8408f4d4ef5f1ed3d93385"
    },
    {
      "path": "Dockerfile",
      "mode": "100644",
      "type": "blob",
      "sha": "8099002c94df984590df75177bf61699816b44ac",
      "size": 337,
      "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/blobs/8099002c94df984590df75177bf61699816b44ac"
    },
    {
      "path": "Dockerfile.2",
      "mode": "100644",
      "type": "blob",
      "sha": "07c4a4a35efd7700a5456bc570ecf15fd2320ab6",
      "size": 364,
      "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/blobs/07c4a4a35efd7700a5456bc570ecf15fd2320ab6"
    },
    {
      "path": "README.md",
      "mode": "100644",
      "type": "blob",
      "sha": "bc7612c525daebdfb3c2e8b710eb348ff121bef7",
      "size": 176,
      "url": "https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/blobs/bc7612c525daebdfb3c2e8b710eb348ff121bef7"
    }
  ],
  "truncated": false
}
```

    Also, we need the sha value.

4. Create a new commit with the new modify file. Invoke the api `https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/commits` . And pack the request body like below.

```json
{
  "message": "commit with rest api",
  "parents": ["970e75be2dada828114ac59d845a23cde29e90ca"],
  "tree": "a3328d8bc90acbac72992b184ae9298fccbe9535",
  "author": {
    "name": "andy zhangtao",
    "email": "ztao8607@gmail.com"
  }
}
```

    "tree" is the step 3 return `sha` value. "parents" is the step 1 sha value. And this api will return a new sha like `8099002c94df984590df75177bf61699816b44ac` (this sha is a fake value !!!)

5. The last step is update branch info. Invoke api `https://api.github.com/repos/andy-zhangtao/sync-docker-hub/git/refs/heads/main` use `POST` method. Body content is

```json
{
  "sha": "8099002c94df984590df75177bf61699816b44ac"
}
```

    If return "200 OK", that say everything is ok. Otherwise, it means invoke failed.

## How to invoke Telegram Bot API ?

To be continue ...
