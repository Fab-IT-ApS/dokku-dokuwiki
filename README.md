# Dokuwiki dokku deployment

To deploy a dokuwiki on dokku, persistant storage is needed. How you
choose to add this volume is up to you, but the volume should mount in
to `/docker-volume` and contain the dirs: `data`, `conf`, and `plugins`.
These dirs will then be symlinked like:

 - `data -> /app/data`
 - `conf -> /app/conf`
 - `plugins -> /app/lib/plugins`

Once this is done, the site will chown the files to be owned by the user
currently running the website, and then start the apache worker.

### Example volume configuration
Using the plugin: https://github.com/ohardy/dokku-volume

 - Add a volume to the instance with

```bash
dokku volume add dokuwiki /docker-volume
```

 - Find path to the volume

```bash
dokku volume list dokuwiki
```

It gives you something like:
`/home/dokku/.o_volume/data/1a298698-b480-4242-81e9-db98161932b3:/docker-volume`
, and you need the part before the ":".

 - Add directories

```bash
cd /tmp
wget -O latest.tgz http://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
tar -zxvf latest.tgz
cp -r dokuwiki*/data /home/dokku/.o_volume/data/1a298698-b480-4242-81e9-db98161932b3/
cp -r dokuwiki*/conf /home/dokku/.o_volume/data/1a298698-b480-4242-81e9-db98161932b3/
cp -r dokuwiki*/lib/plugins /home/dokku/.o_volume/data/1a298698-b480-4242-81e9-db98161932b3/
rm -r dokuwiki* latest.tgz
```

**NOTE:** If you are migrating an existing dokuwiki, copy in the `data`,
`conf` and `lib/plugins` directories into the created volume (as above).

## Notes first use and upgrading

If you are running this for the first time, or updating to a newer
version of dokuwiki, please run the `update-dokuwiki` script as:

```bash
./update-dokuwiki
```

This will pull the latest stable version, remove the dirs you are
going to symlink in and cleanup.

After this you should check dependencies in the `vendor`-folder and
update these dependency versions in composer.json. If you have updated
any dependencies please download composer as:

```bash
curl -sS https://getcomposer.org/installer | php
```

, and run:

```bash
php composer.phar update
```

, to update the dependencies in the `conposer.lock`-file.

Then, no matter if you updated or not, add all files to the git repo and
commit and push them to the server with:

```bash
git add .
git commit -m "Update dokuwiki to version x.y.z"
git push
git push dokku master
```
