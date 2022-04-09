# GitLFS-Issue4928-Test

Repository for testing [Git LFS Issue #4928](https://github.com/git-lfs/git-lfs/issues/4928)

## Troubleshooting
- Create an executible file called `scta` with the following contents:
```
    #!/bin/sh

    echo "Standalone Custom Transfer Agent was called" >> ~/scta.log
```
- Create the following entries in your git config:
```
    [lfs "https://test.example.com"]
      standalonetransferagent = scta
    [lfs "customtransfer.scta"]
      path = [path to the scta executible]
      direction = both
```
- Clone this repo
- Create a non-zero-byte file with the extension `.lfsfile`
- `git add` and `git commit` this file
- Attempt a `git push`
- Check for the existence of `~/scta.log`

In my own testing, this file was never created during the push, when I believe it should have been. Also, the `git-lfs` process seems to `POST` to the `/objects/batch` endpoint of `lfs.url` by itself. I believe it shouldn't be doing this, as there is a standalone custom transfer agent configured for that same URL.

Below is the `GIT_TRACE`, `GIT_TRANSFER_TRACE`, and `GIT_CURL_VERBOSE` output from my own testing:
```
GitLFS-Issue4928 % git add random-binary.lfsfile
GitLFS-Issue4928 % git commit -m "test push of LFS file from local"            
[main 47475f8] test push of LFS file from local
 1 file changed, 3 insertions(+)
 create mode 100644 random-binary.lfsfile
GitLFS-Issue4928 % git config --get-urlmatch lfs.standalonetransferagent https://test.example.com
scta
GitLFS-Issue4928 % GIT_TRACE=1 GIT_TRANSFER_TRACE=1 GIT_CURL_VERBOSE=1 git push
14:14:55.542763 exec-cmd.c:139          trace: resolved executable path from Darwin stack: /usr/local/bin/git
14:14:55.543269 exec-cmd.c:238          trace: resolved executable dir: /usr/local/bin
14:14:55.544019 git.c:439               trace: built-in: git push
14:14:55.547125 run-command.c:663       trace: run_command: GIT_DIR=.git git-remote-https origin https://github.com/GarretFarrell/GitLFS-Issue4928.git
14:14:55.567742 exec-cmd.c:139          trace: resolved executable path from Darwin stack: /usr/local/libexec/git-core/git-remote-https
14:14:55.569531 exec-cmd.c:238          trace: resolved executable dir: /usr/local/libexec/git-core
* Couldn't find host github.com in the .netrc file; using defaults
*   Trying 140.82.121.3...
* TCP_NODELAY set
* Connected to github.com (140.82.121.3) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* SSL connection using TLSv1.2 / ECDHE-ECDSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=US; ST=California; L=San Francisco; O=GitHub, Inc.; CN=github.com
*  start date: Mar 15 00:00:00 2022 GMT
*  expire date: Mar 15 23:59:59 2023 GMT
*  subjectAltName: host "github.com" matched cert's "github.com"
*  issuer: C=US; O=DigiCert Inc; CN=DigiCert TLS Hybrid ECC SHA384 2020 CA1
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fa2d8027200)
> GET /GarretFarrell/GitLFS-Issue4928.git/info/refs?service=git-receive-pack HTTP/2
Host: github.com
User-Agent: git/2.24.3 (Apple Git-128)
Accept: */*
Accept-Encoding: deflate, gzip
Pragma: no-cache

* Connection state changed (MAX_CONCURRENT_STREAMS == 100)!
< HTTP/2 401 
< server: GitHub Babel 2.0
< content-type: text/plain
< content-security-policy: default-src 'none'; sandbox
< content-length: 26
< www-authenticate: Basic realm="GitHub"
< x-frame-options: DENY
< x-github-request-id: EE85:F001:8E75EA:95CDDB:6250354F
< 
* Connection #0 to host github.com left intact
14:14:55.985747 run-command.c:663       trace: run_command: 'git credential-osxkeychain get'
14:14:55.995071 exec-cmd.c:139          trace: resolved executable path from Darwin stack: /usr/local/libexec/git-core/git
14:14:55.995626 exec-cmd.c:238          trace: resolved executable dir: /usr/local/libexec/git-core
14:14:55.996268 git.c:702               trace: exec: git-credential-osxkeychain get
14:14:55.996284 run-command.c:663       trace: run_command: git-credential-osxkeychain get
* Found bundle for host github.com: 0x7fa2d6a19be0 [can multiplex]
* Re-using existing connection! (#0) with host github.com
* Connected to github.com (140.82.121.3) port 443 (#0)
* Server auth using Basic with user [my user number]
* Using Stream ID: 3 (easy handle 0x7fa2d8027200)
> GET /GarretFarrell/GitLFS-Issue4928.git/info/refs?service=git-receive-pack HTTP/2
Host: github.com
Authorization: Basic [my access key]
User-Agent: git/2.24.3 (Apple Git-128)
Accept: */*
Accept-Encoding: deflate, gzip
Pragma: no-cache

< HTTP/2 200 
< server: GitHub Babel 2.0
< content-type: application/x-git-receive-pack-advertisement
< content-security-policy: default-src 'none'; sandbox
< expires: Fri, 01 Jan 1980 00:00:00 GMT
< pragma: no-cache
< cache-control: no-cache, max-age=0, must-revalidate
< vary: Accept-Encoding
< x-frame-options: DENY
< x-github-request-id: EE85:F001:8E7627:95CE10:6250354F
< 
* Connection #0 to host github.com left intact
14:14:56.267032 run-command.c:663       trace: run_command: 'git credential-osxkeychain store'
14:14:56.274314 exec-cmd.c:139          trace: resolved executable path from Darwin stack: /usr/local/libexec/git-core/git
14:14:56.274906 exec-cmd.c:238          trace: resolved executable dir: /usr/local/libexec/git-core
14:14:56.275587 git.c:702               trace: exec: git-credential-osxkeychain store
14:14:56.275607 run-command.c:663       trace: run_command: git-credential-osxkeychain store
14:14:56.387731 run-command.c:663       trace: run_command: .git/hooks/pre-push origin https://github.com/GarretFarrell/GitLFS-Issue4928.git
14:14:56.396635 exec-cmd.c:139          trace: resolved executable path from Darwin stack: /usr/local/libexec/git-core/git
14:14:56.397155 exec-cmd.c:238          trace: resolved executable dir: /usr/local/libexec/git-core
14:14:56.397833 git.c:702               trace: exec: git-lfs pre-push origin https://github.com/GarretFarrell/GitLFS-Issue4928.git
14:14:56.397849 run-command.c:663       trace: run_command: git-lfs pre-push origin https://github.com/GarretFarrell/GitLFS-Issue4928.git
14:14:56.409301 trace git-lfs: exec: git 'version'
14:14:56.414656 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'remote' '-v'
14:14:56.419520 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'remote'
14:14:56.424223 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'rev-parse' 'HEAD' '--symbolic-full-name' 'HEAD'
14:14:56.428576 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'rev-parse' '--git-dir' '--show-toplevel'
14:14:56.432570 trace git-lfs: exec: git 'config' '-l'
14:14:56.436872 trace git-lfs: exec: git 'rev-parse' '--is-bare-repository'
14:14:56.441024 trace git-lfs: exec: git 'config' '-l' '-f' '/Users/garret/test/GitLFS-Issue4928/.lfsconfig'
14:14:56.446878 trace git-lfs: pre-push: refs/heads/main 47475f8891a73474e1ff9d4b01da91311644604f refs/heads/main d2eec8bb86fdcff095203aec714eb0d9f5f8d781
14:14:56.447088 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'show-ref'
14:14:56.451900 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'ls-remote' '--heads' '--tags' '-q' 'origin'
14:14:56.850587 trace git-lfs: exec: /usr/bin/security 'list-keychains'
14:14:56.891334 trace git-lfs: exec: /usr/bin/security 'find-certificate' '-a' '-p' '-c' 'test.example.com' '/Library/Keychains/System.keychain'
14:14:56.945802 trace git-lfs: HTTP: POST https://test.example.com/locks/verify
> POST /locks/verify HTTP/1.1
> Host: test.example.com
> Accept: application/vnd.git-lfs+json; charset=utf-8
> Content-Length: 34
> Content-Type: application/vnd.git-lfs+json; charset=utf-8
> User-Agent: git-lfs/2.13.1 (GitHub; darwin amd64; go 1.15.5; git e896fc7a)
> 
Remote "origin" does not support the LFS locking API. Consider disabling it with:
  $ git config lfs.https://test.example.com.locksverify false
14:14:56.992581 trace git-lfs: tq: running as batched queue, batch size of 100
14:14:56.993319 trace git-lfs: run_command: git rev-list --objects --ignore-missing --stdin --
14:14:56.995203 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'cat-file' '--batch-check'
14:14:56.996886 trace git-lfs: exec: git '-c' 'filter.lfs.smudge=' '-c' 'filter.lfs.clean=' '-c' 'filter.lfs.process=' '-c' 'filter.lfs.required=false' 'rev-parse' '--git-common-dir'
14:14:57.006418 trace git-lfs: tq: sending batch of size 1                                                                                                                                                                                                    
14:14:57.009202 trace git-lfs: api: batch 1 files
14:14:57.009340 trace git-lfs: HTTP: POST https://test.example.com/objects/batch
> POST /objects/batch HTTP/1.1
> Host: test.example.com
> Accept: application/vnd.git-lfs+json; charset=utf-8
> Content-Length: 220
> Content-Type: application/vnd.git-lfs+json; charset=utf-8
> User-Agent: git-lfs/2.13.1 (GitHub; darwin amd64; go 1.15.5; git e896fc7a)
> 
14:14:57.018500 trace git-lfs: api error: Post "https://test.example.com/objects/batch": dial tcp: lookup test.example.com: no such host
Uploading LFS objects:   0% (0/1), 0 B | 0 B/s, done.
batch response: Post "https://test.example.com/objects/batch": dial tcp: lookup test.example.com: no such host
error: failed to push some refs to 'https://github.com/GarretFarrell/GitLFS-Issue4928.git'
* Closing connection 0
```
