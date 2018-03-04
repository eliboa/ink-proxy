<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="Ink Proxy">
    <main>
        <div v-if="step == 0" class="doc">
          <div class="title">Getting Started</div>
          <p>
            To retrieve your iksm session token, you need :<br>
            <li>An Android or iOS device with "Nintendo Switch Online" app installed (but not running, kill active process if needed) </li>
            <li>Both this computer and your device connected to the same router</li><br>
          </p>
          <button @click="next()">I'm ready</button><br><br>
          <div class="title">Service status</div>
          <p>
            Name : <a @click="open('https://mitmproxy.org/')" href="#">mitm proxy</a><br>
            Status : {{ status }}
          </p>
        </div>
        <div v-if="step == 1" class="doc">
          <p>
            On your phone, go to <b>Settings</b> > <b>Wi-Fi</b> then <b>Modify network</b> (on your active connection) :<br><br>
            <div class="center"><img id="step_img" src="~@/assets/screenshot_01.jpg"></div>
          </p>
          <div id="buttons">
            <button @click="prev()" class="left">Previous step</button>
            <button @click="next()" class="right">Next step</button>
          </div>
          <br><br>
        </div>
        <div v-if="step == 2" class="doc">
          <p>
            In <b>Advanced options</b> > <b>Proxy</b> select <b>Manual</b> :<br><br>
            <div class="center"><img id="step_img" src="~@/assets/screenshot_02.jpg"></div>
          </p>
          <div id="buttons">
            <button @click="prev()" class="left">Previous step</button>
            <button @click="next()" class="right">Next step</button>
          </div>
          <br><br>
        </div>
        <div v-if="step == 3" class="doc">
          <p>
            Set <b>Proxy hostname</b> to <b>{{ ip_address }}</b> (it's your computer's IP address). Port is  <b>8080</b>.<br><br>
            <div class="center"><img id="step_img" src="~@/assets/screenshot_03.jpg"></div>
          </p>
          <div id="buttons">
            <button @click="prev()" class="left">Previous step</button>
            <button @click="next()" class="right">Next step</button>
          </div>
          <br><br>
        </div>
        <div v-if="step == 4" class="doc">
          <p>
            <b>Save</b> changes. Then, open this url in Chrome/Safari: <b>mitm.it</b><br>
            <b>Install</b> the android or apple certificate (you can skip this step if you previously installed the certificate). <br><br>
            <div class="center"><img id="step_img" src="~@/assets/screenshot_04.jpg"></div>
          </p>
          <div id="buttons">
            <button @click="prev()" class="left">Previous step</button>
            <button @click="next()" class="right">Next step</button>
          </div>
          <br><br>
        </div>
        <div v-if="step == 5" class="doc">
          <p>
            Open <b>Nintendo Switch Online</b> app on your device. Click on the "Splatoon 2" logo and wait.<br>
            The token should pop in a moment (otherwise an error occurred) <br><br>
            <div class="center"><img id="step_img" src="~@/assets/screenshot_05.jpg"></div>
          </p>
          <div id="buttons">
            <button @click="prev()" class="left">Previous step</button>
          </div>
          <br><br>
        </div>
        <div v-if="step == 6" class="doc">
          <div class="title">Token found !</div>
          <p>
            Here is your iksm_session token :<br>
            <div class="token">{{ iksm_session }}</div><br>
          </p>
          <p>
            You can now set back <b>Proxy</b> to <b>None</b> in your <b>Wi-Fi settings</b>.
          </p>
          <br><br>
        </div>
    </main>
    <div id="foot"><a @click="open('https://github.com/eliboa')" href="#">eliboa</a></div>
  </div>
</template>

<script>
  export default {
    name: 'landing-page',
    data () {
      return {
        iksm_session: false,
        ip_address: require('ip').address(),
        proxySatus: 0,
        step: 0
      }
    },
    computed: {
      status: function () {
        switch (this.proxySatus) {
          case -1:
            return 'Error while loading proxy.'
          case 0:
            return 'Proxy is loading...'
          case 1:
            return 'Proxy is running, listening for SplatNet connection... (at ' + this.ip_address + ':8080)'
          case 2:
            return 'Token found !'
          default:
            return ''
        }
      }
    },
    created: function () {
      this.loadProxy()
    },
    methods: {
      open (link) {
        this.$electron.shell.openExternal(link)
      },
      next () {
        this.step++
      },
      prev () {
        this.step--
      },
      loadProxy: function () {
        const fs = require('fs')
        const spawn = require('child_process').spawn
        const child = spawn('resources/mitmdump.exe', ['-w', 'proxy.sniff', '~hq iksm_session'])
        var enc = new TextDecoder()

        child.stdout.on('data', (data) => {
          var stdout = enc.decode(data)
          if (stdout.indexOf('Proxy server listening at') > -1) {
            this.proxySatus = 1
          }
          if (stdout.indexOf('app.splatoon2.nintendo.net') > -1 && this.proxySatus !== 3) {
            this.proxySatus = 2
            console.log('Splatnet connection detected')
            // Stop mitm process
            child.kill()
            // Read output file
            fs.readFile('proxy.sniff', (err, data) => {
              if (err) console.log('ERR=' + err)
              var matches = enc.decode(data).match(/iksm_session=([0-9A-Fa-f]*)/)
              if (matches !== 'undefined' && matches.length > 1) {
                this.iksm_session = matches[1]
                this.proxySatus = 3
                this.step = 6
              }
            })
          }
        })
        child.stderr.on('data', (data) => {
          this.proxySatus = -1
          console.error(`child stderr:\n${data}`)
        })
        child.on('exit', function (code, signal) {
          console.log('child process exited with ' +
                      `code ${code} and signal ${signal}`)
        })
      }
    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body { font-family: 'Source Sans Pro', sans-serif; }

  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    padding: 40px 80px;
    width: 100vw;
  }

  #logo {
    height: auto;
    margin-bottom: 20px;
    width: 420px;
  }

  main {
    display: flex;
    justify-content: space-between;
  }

  main > div { flex-basis: 100%; }

  .left-side {
    display: flex;
    flex-direction: column;
  }

  .welcome {
    color: #555;
    font-size: 23px;
    margin-bottom: 10px;
  }

  .title {
    color: #2c3e50;
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 6px;
    font-family: "Splatoon2";
  }

  .title.alt {
    font-size: 18px;
    margin-bottom: 10px;
  }

  .doc p {
    color: black;
    margin-bottom: 10px;
  }

  .doc button {
    font-size: .8em;
    cursor: pointer;
    outline: none;
    padding: 0.75em 2em;
    border-radius: 2em;
    display: inline-block;
    color: #fff;
    background-color: #4fc08d;
    transition: all 0.15s ease;
    box-sizing: border-box;
    border: 1px solid #4fc08d;
    font-family: "Splatoon2";
  }

  .doc button.alt {
    color: #42b983;
    background-color: transparent;
  }

  #step_img {
    height: 40vh;
  }

  #buttons {
    display: flex;
    width: 100%;
  }

  #buttons .left {
    margin: auto;
    margin-left: 0;
  }

  #buttons .right {
    margin: auto;
    margin-right: 0;
  }

  .token {
    padding: 0.75em 2em;
    font-size: 1.3em;
    border-radius:2em;
    background-color: #4fc08d;
    transition: all 0.15s ease;
    text-align: center;
    width: 80%;
    margin: auto;
    font-family: "Splatoon2";
    color: #fff;
  }
  .center {
    width: 100%;
    text-align: center;
  }
   #foot {
     position: absolute;
     right:0;
     bottom:0;
     padding: 10px;
     font-family: "Splatoon2";
     color: grey;
   }

   #foot a {
     color: grey;
     text-decoration: none;
   }

</style>
