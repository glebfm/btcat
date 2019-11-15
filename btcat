#!/usr/bin/env python

#http://www.rasterbar.com/products/libtorrent/manual.html
import libtorrent as lt
import time
import types
import sys
import Tkinter
from subprocess import *
import thread
import socket
import os
import urllib

import hashlib
def md5(s):
    m = hashlib.md5()
    m.update(s)
    return m.hexdigest()

strout = None
played=0

def compact(s):
    inarow = 1
    mx = 5
    i = 1
    while i < len(s):
        if s[i] != s[i-1]:
            if inarow > mx:
                ins = '['+s[i-1]+'*'+str(inarow)+']'
                s = s[0:i-inarow]+ins+s[i:]
                i = i - inarow + len(ins)
            inarow = 1
        else:
            inarow=inarow+1
        i = i+1
    if inarow > mx:
        ins = '['+s[i-1]+'*'+str(inarow)+']'
        s = s[0:i-inarow]+ins+s[i:]
        i = i - inarow + len(ins)
    return s

def printstatus():
    state_str = ['queued', 'checking', 'downloading metadata', 'downloading', 'finished', 'seeding', 'allocating', 'checking fastresume']
    s = h.status()
    #print >> sys.stderr,'%.2f%% complete (down: %.1f kb/s up: %.1f kB/s peers: %d seeds: %d) %s\n' % (s.progress * 100, s.download_rate / 1000, s.upload_rate / 1000, s.num_peers, s.num_seeds,state_str[s.state]),

    #if s.state == 4:
    #    break
    sys.stdout.flush()
    l = ''
    i = 0
    completed = 0
    for p in s.pieces:
        if i >= piecestart and i <= pieceend:
            if p == True:
                l = l + '1'
                completed = completed+1
            if p == False:
                l = l + '0'
        i = i+1
    progress = 0
    if i!=0:
        progress = 1.*completed/i
    #print >> sys.stderr,l
    #s = '%.2f%% down: %.1f kb/s up: %.1f kB/s peers: %d seeds: %d pieces: %s' % (progress * 100, s.download_rate / 1000, s.upload_rate / 1000, s.num_peers, s.num_seeds, compact(l))
    s = 'completed: %.2f%% down: %.1f kb/s up: %.1f kB/s peers: %d seeds: %d' % (progress * 100, s.download_rate / 1000, s.upload_rate / 1000, s.num_peers, s.num_seeds)
    print >> sys.stderr,s+'\n',

def addnewpieces():
    prio = h.piece_priorities()
    s = h.status()
    downloading = 0
    if len(s.pieces) == 0:
        return
    for piece in range(piecestart,pieceend+1):
        if prio[piece] != 0 and s.pieces[piece]==False:
            die2()
            downloading = downloading+1
    for piece in range(piecestart,pieceend+1):
        if prio[piece] == 0 and downloading < piecesperite:
            die2()
                        #if piece < piecestart+100:
                        
            #print >> sys.stderr,'downloading piece ',piece
            h.piece_priority(piece,1)
            downloading = downloading+1
    for piece in range(piecestart,pieceend+1):
        if prio[piece] != 0 and s.pieces[piece]==False:
            die2()
            #print >> sys.stderr,'high prio ',piece
            #h.piece_priority(piece,7)
            break

kill = False
def die():
    global kill
    if kill == True:
        #print >> sys.stderr,'exiting thread 1'
        kill = False
        thread.exit()

kill2 = False
def die2():
    global kill2
    if kill2 == True:
        #print >> sys.stderr,'exiting thread 2'
        kill2 = False
        thread.exit()

cache = {}
def getpiece(i):
    global cache
    if i in cache:
        ret = cache[i]
        cache[i] = 0
        return ret
    while True:
        s = h.status()
        if len(s.pieces)==0:
            break
        if s.pieces[i]==True:
            break
        time.sleep(.1)
        die()
    h.read_piece(i)
    while True:
        #printstatus()
        #addnewpieces()
        piece = ses.pop_alert()
        if isinstance(piece, lt.read_piece_alert):
            if piece.piece == i:
                #sys.stdout.write(piece.buffer)
                return piece.buffer
            else:
                #print >> sys.stderr,'store somewhere'
                cache[piece.piece] = piece.buffer
            break
        time.sleep(.1)
        die()

def runmplayer():
    time.sleep(1)
    #os.system("ffplay http://127.0.0.1:50008")
    os.system("player.bat")
    #os.system("vlc http://127.0.0.1:50008")
    #os.system("vlc play.m3u")
    #os.system("mplayer -fs http://127.0.0.1:50008")
    #os.system("D:/Python26/mplayer -cache 100 -fs http://127.0.0.1:50008")

def read2(s):
    conn, addr = s.accept()
    #print 'Connected by 2', addr
    data = conn.recv(1024)
    print data

    
completed = False
def writethread():
    global completed,played,subprocess,kill
    stream = 0
    conn = 0
    for piece in range(piecestart,pieceend+1):
        buf=getpiece(piece)
        played=piece-piecestart
        if piece==piecestart:
            buf = buf[offset1:]
        if piece==pieceend:
            buf = buf[:offset2]
        if outputcmd=='-':
            stream = sys.stdout
        elif outputcmd=='http':
            if conn == 0:
                HOST = '127.0.0.1'                 # Symbolic name meaning all available interfaces
                PORT = 50008              # Arbitrary non-privileged port
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                s.bind((HOST, PORT))
                s.listen(1)
                #print "mplayer http://127.0.0.1:"+str(PORT)
                #os.system("mplayer http://127.0.0.1:"+str(PORT))
                #os.spawnl(os.P_NOWAIT, "mplayer http://127.0.0.1:"+str(PORT))
                thread.start_new_thread(runmplayer,())
                conn, addr = s.accept()
                print 'Connected by', addr
                data = conn.recv(1024)
                print data
                #thread.start_new_thread(read2,(s,))
                #conn.send('HTTP/1.1 206 Partial Content\r\nContent-Type: video/mp4\r\n\r\n')
                conn.send('HTTP/1.1 200 OK\r\nContent-Type: video/mp4\r\n\r\n')
                #conn.send('HTTP/1.1 200 OK\r\nContent-Type: video/x-msvideo\r\n\r\n')
                #conn.send('HTTP/1.1 206 Partial Content\r\nDate: Sun, 18 Sep 2011 13:34:12 GMT\r\nServer: Apache/2.2.6 (Unix) mod_ssl/2.2.6 OpenSSL/0.9.7a mod_bwlimited/1.4 FrontPage/5.0.2.2635 mod_auth_passthrough/2.1 PHP/5.2.4\r\nLast-Modified: Thu, 03 Mar 2005 21:18:36 GMT\r\nETag: "5814d8-663c88-2c3d2300"\r\nAccept-Ranges: bytes\r\nContent-Length: 6011062\r\nContent-Range: bytes 689106-6700167/6700168\r\nContent-Type: video/x-msvideo\r\n\r\n')

        else:
            if stream == 0:
                subprocess = Popen(outputcmd.split(' '), stdin=PIPE)
                stream = subprocess.stdin
        try:
            #if piece == piecestart+1:
            #    time.sleep(100)
            if stream != 0:
                stream.write(buf)
            if conn != 0:
                #print >> sys.stderr, 'played',piece
                #print >> sys.stderr, 'output',piece,len(buf)
                r=conn.sendall(buf)
                #print 'ret',r
        except Exception, err:
            ses.remove_torrent(h)
            completed = True
            #print 'exiting thread 1 (exception)'
            kill = False
            kill2 = True
            #log.log('')
            thread.exit()
            #exit(0)
        time.sleep(.1)
        die()
    ses.remove_torrent(h)
    completed = True

def cancel():
    global kill,kill2
    #ses.abort()
    #ses.remove_torrent(h)
    kill = True
    kill2 = True
    if subprocess != None:
        subprocess.terminate()
    h.pause()

def start(torrent,fileid,outdir,_outputcmd):
    global ses,h,piecestart,pieceend,offset1,offset2,piecesperite,outputcmd,kill,kill2,subprocess,completed,played
    played=0
    kill = False
    kill2 = False
    subprocess = None
    completed = False
    outputcmd=_outputcmd
    info = lt.torrent_info(torrent)
    piecesperite = 40*1024*1024/info.piece_length() # 40 MB
    #print >> sys.stderr, 'piecesperite',piecesperite
    #print >> sys.stderr, 'info.piece_length()',info.piece_length()
    sizes = []
    i = 0
    for f in info.files():
        piecestart = f.offset/info.piece_length()
        pieceend = (f.offset+f.size)/info.piece_length()
        sizes.append(f.size)
        print >> sys.stderr, i,f.path,f.size,f.offset,piecestart,pieceend
        i=i+1
    if fileid == 'list':
        return
    if fileid == 'max':
        fileid = sizes.index(max(sizes))
    else:
        fileid = int(fileid)

    f = info.files()[fileid]
    #print >> sys.stderr, f.path
    piecestart = f.offset/info.piece_length()
    pieceend = (f.offset+f.size)/info.piece_length()
    offset1 = f.offset%info.piece_length() #how many bytes need to be removed from the 1st piece
    offset2 = ((f.offset+f.size)%info.piece_length()) #how many bytes need we keep from the last piece
    #print >> sys.stderr,piecestart,pieceend,offset1,offset2,info.piece_length()
    #print >> sys.stderr,(pieceend-piecestart+1)*info.piece_length()-(offset1+offset2),f.size
    ses = lt.session()

    state = None
    #state = lt.bdecode(open(state_file, "rb").read())
    ses.start_dht(state)
    ses.add_dht_router("router.bittorrent.com", 6881)
    ses.add_dht_router("router.utorrent.com", 6881)
    ses.add_dht_router("router.bitcomet.com", 6881)
    ses.listen_on(6881, 6891)
    ses.set_alert_mask(lt.alert.category_t.storage_notification)
    h = ses.add_torrent({'ti': info, 'save_path': outdir})
    for i in range(info.num_pieces()):
        h.piece_priority(i,0)
    #print >> sys.stderr,'starting', h.name()
    for i in range(piecestart,piecestart+piecesperite):
        if i <= pieceend:
            h.piece_priority(i,7)
            #print >> sys.stderr,'downloading piece '+str(i)
    h.set_sequential_download(True)
    thread.start_new_thread(writethread,())
    while not completed:
        printstatus()
        addnewpieces()
        time.sleep(1)
        die2()
    #print 'exiting thread 2 (end)'

def main():
    if len(sys.argv) == 1:
        print 'Usage: btcat TORRENTFILE|TORRENTURL [FILEID]'
        exit(0)
    torrent=sys.argv[1]
    fileid='list'
    #if len(sys.argv) > 2:
        #fileid=int(sys.arSgv[2])
    outdir='/tmp'
    #outdir='/media/data/btcat'
    outputcmd='http'
    outputcmd='vlc -'
    outputcmd='mplayer -fs -'
    outputcmd='mplayer -aspect 16:9 -fs -'
    outputcmd='-'
    #fileid='max'
    if len(sys.argv)>2:
        fileid=sys.argv[2]
    if len(sys.argv)>3:
        outdir=sys.argv[3]
    if len(sys.argv)>4:
        outputcmd=sys.argv[4]
    if len(torrent)>4 and torrent[0:4] == 'http':
        dest = outdir+'/'+md5(torrent)
        urllib.urlretrieve(torrent, dest)
        torrent = dest
    start(torrent,fileid,outdir,outputcmd)

if __name__ == "__main__":
    main()

