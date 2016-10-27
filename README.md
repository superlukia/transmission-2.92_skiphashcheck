# transmission-2.92_skiphashchek
## Add feature "skip hash check" for transmission 2.92

"skip hash check" is a very useful feature for seeding identical big torrents on different sites. It exists in uTorrent but the offical release of transmission doesn't support this function.

### Modify code yourself

In this project, I modified less than 10 lines of code to add this feature for transmission-2.92. It's very simple:

you can download source code from the offical site,modify the following changes and compile yourself. then replace `/usr/bin/transmission-daemon` with `transmission-daemon` in `daemon` directory after `make`

diff libtransmission/rpcimpl.c

    388a389
    >   skiphash();

diff libtransmission/verify.c

    41a42,46
    > static bool skiphashcheck=false;
    > void skiphash(){
    >     skiphashcheck=true;
    > }
    > 
    76c81
    <       if (filePos == 0 && fd == TR_BAD_SYS_FILE && fileIndex != prevFileIndex)
    ---
    >       if (!skiphashcheck && filePos == 0 && fd == TR_BAD_SYS_FILE && fileIndex != prevFileIndex)
    95c100
    <           if (tr_sys_file_read_at (fd, buffer, bytesThisPass, filePos, &numRead, NULL) && numRead > 0)
    ---
    >           if (!skiphashcheck && tr_sys_file_read_at (fd, buffer, bytesThisPass, filePos, &numRead, NULL) && numRead > 0)
    119c124
    <           hasPiece = !memcmp (hash, tor->info.pieces[pieceIndex].hash, SHA_DIGEST_LENGTH);
    ---
    >           hasPiece = skiphashcheck || !memcmp (hash, tor->info.pieces[pieceIndex].hash, SHA_DIGEST_LENGTH);
    156a162,165
    >   if(skiphashcheck){
    >       skiphashcheck=false;
    >       tr_logAddTorInfo (tor, "%s", _("skip hash check"));
    >   }

this patch is for transmission-2.92, but for other version I think it will be similar

### Download and compile directly
download the project and:

    $ cd transmission-2.92
    $ ./configure
    $ make
    $ sudo make install

if you have installed transmission-2.92, replace `/usr/bin/transmission-daemon` with `transmission-daemon` in `daemon` directory after `make`.


### How to use
In the web interface: `http://ip:9091/transmission/web/`.
right click on ANY torrent, click `Ask tracker for more peers` and the CURRENT verifying torrent will be skipped for hash check.


* DON'T replace file if version of transmission installed is other than 2.92. 
* Make a backup of `/usr/bin/transmission-daemon` before replacing. 
* I have only tested transmission-2.92 on Centos 6.
* DON'T use this feature for cheating, you will be caught.
