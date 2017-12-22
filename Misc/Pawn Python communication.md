# Pawn-Python Communication
This is just a "really bad" idea popped in mind.There is alot better methods like using redis or embedding python as a plugin (python.h) so i don't recommend to use this method in production environment.Well the the idea was communcation of python with pawn using a web server as a medium.We can set up a webserver in python using any framework like Flask and communcate it with SAMP's a_http library.Some of the examples are give below:

## Youtube 2 Mp3
### Python code
```Python
from flask import Flask,send_from_directory
import youtube_dl
app  = Flask(__name__)

@app.route("/audio/<string:vid>")

def audio(vid):
    ydl_opts = {
        'format': 'bestaudio/best',
        'outtmpl': '/up/'+vid+'.mp3',
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '192',
            
        }],
    }
    with youtube_dl.YoutubeDL(ydl_opts) as ydl:
        ydl.download(['http://www.youtube.com/watch?v='+vid])    
    return "/up/"+vid+".mp3"

@app.route('/up/<path:path>')
def send(path):
    return send_from_directory('up', path)

app.run("localhost",5000,debug=True,threaded=True)
```
### Pawn Code

```Pawn
#include<a_samp>
#include<a_http>
#include<zcmd>

CMD:start(playerid,params[])
{
	HTTP(playerid, HTTP_GET, "localhost:5000/audio/h2XTsWgN0CU", "", "MyHttpResponse");
	return 1;
}

CMD:play(playerid,params[])
{
	new payload[70];
    format(payload,sizeof payload,"localhost:5000/audio/%s",params);
    HTTP(playerid, HTTP_GET, payload, "", "PythonSays");
	return 1;
}

forward PythonSays(index, response_code, data[]);
public PythonSays(index, response_code, data[])
{
	new buf[100];
	format(buf,sizeof buf,"[%d]Python says : %s",response_code,data);
	SendClientMessage(index,-1,buf);
    PlayAudioStreamForPlayer(index, data);
	return 1;
}
````

