<h1>ircII in WebAssembly</h1>

<h2>Introduction</h2>

<p>ircII is a free, open-source Unix IRC and ICB client written in C. Initially released in the late 1980s, it is the oldest IRC client still maintained. <a href="https://en.wikipedia.org/wiki/IrcII" target="_blank">Wikipedia</a>.</p>

<p>This limited emulation of ircII in WebAssembly demonstrates the use of POSIX and similar libraries such as <a href="https://en.wikipedia.org/wiki/Ncurses" target="_blank">ncurses</a> within the <a href="https://emscripten.org/" target="_blank">Emscripten</a> environment.</p>

<p>Advantages of this technology can be found if Emscripten's <a href="https://emscripten.org/docs/api_reference/Filesystem-API.html#FS.init">output</a> could work inline with Progressive Web Applications (PWAs) and relay screen dimensions dynamically. Further, both ircII and ncurses compile sizes are fairly large meaning optimisations to reduce compile output is important.</p>

<p>Furthermore, help files and other extended parts of this emulation such as DCC may not work, however, makes the client fun to tinker with as it functions as a working IRC client.</p>
  
<h2>Installation</h2>

<p><b>Important Note</b>: This build of ircII depends on ncurses for Emscripten, see <a href="https://github.com/jamesbiv/ncurses-emscripten" target="_blank">jamesbiv/ncurses-emscripten</a> to compile and build your own copy or you can use what's stored in the <em>build/</em> directory. Further, be sure to have the absolute path of the <em>build/</em> directory handy to replace the areas marked <b>PATH_TO_NCURSES</b> throughout the below compilation process.</p>

<p>It's recommended to create a separate directory to build this project in. For example <em>ircii-emscripten/</em>.</p>

<h3>Downloading the required files</h3>

<pre>~$ wget http://ircii.warped.com/ircii-20190117.tar.gz</pre>

<pre>~$ wget https://raw.githubusercontent.com/jamesbiv/ircii-emscripten/master/ircii-20190117_emscripten.patch</pre>

<pre>~$ wget https://raw.githubusercontent.com/jamesbiv/ircii-emscripten/master/irc_shell.html</pre>

<h3>Extracting and patching ircII</h3>

<pre>~$ tar -zxvf ircii-20190117.tar.gz</pre>

<p><b>Note</b>: Because this is a source tree level patch make sure the patch is in the directory before the installed directory. For example, if the directory for ircii is <em>/home/user/ircii-emscripten/ircii-20190117</em> then make sure the patch is located at <em>/home/user/ircii-emscripten/ircii-20190117_emscripten.patch</em>.</p>

<pre>~$ patch -p0 < ircii-20190117_emscripten.patch</pre>

<pre>~$ cd ircii-20190117</pre>

<h3>Change configure</h3>

<pre>~$ nano -w ./configure</pre>

<p>Comment out the following line of code using <b>#</b> to reflect the following:</p>

<pre>Line 696
<b>#</b>with_openssl
</pre>

<p>Save and close nano</p>

<h3>Configure and make the Emscripten components</h3>

<p><b>Note</b>: Be sure to replace <b>PATH_TO_NCURSES</b> with the absolute path to the ncurses for emscripten <em>build/</em> directory.</p>

<pre>~$ export CPPFLAGS='-I<b>PATH_TO_NCURSES</b>/include'</pre>

<pre>~$ export LDFLAGS='-L<b>PATH_TO_NCURSES</b>/lib'</pre>

<pre>~$ emconfigure ./configure</pre>

<h3>Change Makefile</h3>

<pre>~$ nano -w ./Makefile</pre>

<pre>Line 81
Remove <b>-lssl -lcrypto</b> from <b>LIBS = ... </b></pre>

<p><b>Note</b>: Be sure to replace <b>PATH_TO_NCURSES</b> with the absolute path to the ncurses for emscripten <em>build/</em> directory.</p>

<pre>Line 320
Add (append) <b>-IPATH_TO_NCURSES/include</b> to the end of <b>INCLUDES = -I ...</b></pre>

<p>Save and close nano</p>

<h3>Change defs.h</h3>

<pre>~$ nano -w ./defs.h</pre>

<p>Change the following segments to reflect the following:</p>
<pre>Line 44
/* #undef HAVE_GETNAMEINFO */</pre>
<p>To</p>
<pre>Line 44
#define HAVE_GETNAMEINFO 1</pre>

<p>Change</p>
<pre>Line 119
#define HAVE_SYS_FCNTL_H 1</pre>
<p>To</p>
<pre>Line 119
/* #undef HAVE_SYS_FCNTL_H */</pre>

<p>Change</p>
<p><b>Note</b>: <em>NON_BLOCKING_CONNECTS</em> should be declared but it's best to check it anyway.</p>
<pre>Line 180
/* #undef NON_BLOCKING_CONNECTS */</pre>
<p>To</p>
<pre>Line 180
#define NON_BLOCKING_CONNECTS 1</pre>

<p>Change</p>
<pre>Line 221
#define USE_OPENSSL 1</pre>
<p>To</p>
<pre>Line 221
/* #undef USE_OPENSSL */</pre>

<p>Save and close nano<p>

<h3>Make irc</h3>

<pre>~$ emmake make irc</pre>

<pre>~$ cp irc ../irc.bc</pre>

<pre>~$ cd ..</pre>

<h3>Final build</h3>

<p><b>Note</b>: Be certain to include the terminfo database files with <em>--preload-file</em>. You can just include the file(s) that you need or include them all but you'll save about 3MB if you strip them down to the ones you just need, in this case we're using <em>xterm-new</em>.</p>

<p><b>Note</b>: Be sure to replace <b>PATH_TO_NCURSES</b> with the absolute path to the ncurses for emscripten <em>build/</em> directory.</p>

<pre>emcc -o irc.html irc.bc -O3 -g2 --preload-file <b>PATH_TO_NCURSES</b>/share/terminfo@/home/web_user/.terminfo -s WASM=1 -s FORCE_FILESYSTEM=1 --shell-file ./irc_shell.html -s EMTERPRETIFY=1 -s EMTERPRETIFY_ASYNC=1 -s EMTERPRETIFY_WHITELIST='["_main", "_irc_io"]' -s TOTAL_MEMORY=32Mb --no-heap-copy -s ASSERTIONS=1</pre>

<h2>Installing XTerm.js</h2>

<p>Xterm.js is a great library that emulates a terminal very quickly and works well with ncurses.</p>

<p>You'll need to download a copy from <a href="https://github.com/xtermjs/xterm.js" target="_blank">xtermjs/xterm.js</a> or you can use the <em>.min</em> versions that I have uploaded on the <a href="http://68.183.3.42/ircii-emscripten/irc.html" target="_blank">demo site</a>.</p>

<h2>Socket Setup - WebSockets</h2>

<p>In order to connect to a working IRC server you’ll need to use something like <a href="https://github.com/novnc/websockify" target="_blank">WebSockify</a>. Ideally WebSocket support for IRCD would be great but for now this does the job fine.</p>

<p><b>Note</b>: I suggest if you're using WebSockify filter the incoming port from the public as some IRC networks / servers may have proxy protection and will ban you from their server(s) if discovered.</p>

<h2>Demo Site</h2>

<p>Please see <a href="http://68.183.3.42/ircii-emscripten/irc.html" target="_blank">http://68.183.3.42/ircii-emscripten/irc.html</a> for a working demo.</p>

<h2 name="finalconsiderations">Final Considerations</h2>

<p>IrcII for Emscripten was a fun project and opens up many possibilities and areas for improvement. Should a future project arise from this patch level I believe forking and customising the current ircii build would be the best scenario.</p>

<p>Also, since Emscripten relies on <a href="https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API" target="_blank">WebSockets</a> to handle communications disabling OpenSSL at a C level is required. TLS/SSL communications are evoked using <b>wss://</b> as the WebSocket connection string.</p>

<p>Further, since Emscripten's WebSockets use <em>binary</em> protocol this also needs to be enabled on the service acting as the server.</p>
