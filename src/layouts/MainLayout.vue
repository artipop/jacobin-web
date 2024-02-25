<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title> Quasar App </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header> Essential Links </q-item-label>

        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import EssentialLink from "components/EssentialLink.vue";
import { fs } from "memfs";
import "../stores/wasm_exec.js";
import * as stream from "stream";

const javaHome = "/yyy" + Math.round(Math.random() * 100);
fs.mkdirSync(javaHome, (error) => {});
fs.mkdirSync(javaHome + "/jmods", (error) => {});

fs.writeFileSync(
  javaHome + "/release",
  `JAVA_VERSION="AAA"
JAVA_VERSION_DATE="2023-04-18"
MODULES="java.base java.compiler java.datatransfer java.xml java.prefs java.desktop java.instrument java.logging java.management java.security.sasl java.naming java.rmi java.management.rmi java.net.http java.scripting java.security.jgss java.transaction.xa java.sql java.sql.rowset java.xml.crypto java.se java.smartcardio jdk.accessibility jdk.internal.jvmstat jdk.attach jdk.charsets jdk.compiler jdk.crypto.ec jdk.crypto.cryptoki jdk.dynalink jdk.internal.ed jdk.editpad jdk.hotspot.agent jdk.httpserver jdk.incubator.foreign jdk.incubator.vector jdk.internal.le jdk.internal.opt jdk.internal.vm.ci jdk.internal.vm.compiler jdk.internal.vm.compiler.management jdk.jartool jdk.javadoc jdk.jcmd jdk.management jdk.management.agent jdk.jconsole jdk.jdeps jdk.jdwp.agent jdk.jdi jdk.jfr jdk.jlink jdk.jpackage jdk.jshell jdk.jsobject jdk.jstatd jdk.localedata jdk.management.jfr jdk.naming.dns jdk.naming.rmi jdk.net jdk.nio.mapmode jdk.random jdk.sctp jdk.security.auth jdk.security.jgss jdk.unsupported jdk.unsupported.desktop jdk.xml.dom jdk.zipfs"
`
);
let readFileSync = fs.readFileSync(javaHome + "/release", "utf8");
console.log(readFileSync);

// read MODULES (java.base, etc.) from the top and fetch using `fetch("myhost/jmods/{name}.jmod")`
//  and then put it to memfs

const responseToReadable = (response) => {
  const reader = response.body.getReader();
  const rs = new stream.Readable();
  rs._read = async () => {
    const result = await reader.read();
    if (!result.done) {
      rs.push(Buffer.from(result.value));
    } else {
      rs.push(null);
    }
  };
  return rs;
};

// TODO Use modern approach to fetch https://stackoverflow.com/a/73879265/23340352
const a = ["java.base", "java.compiler"];
for (const moduleName of a) {
  const response = await fetch(
    "http://localhost:8080/jmods/" + moduleName + ".jmod"
  );
  responseToReadable(response).pipe(
    fs.createWriteStream(javaHome + "/jmods/" + moduleName + ".jmod")
  );
}

// TODO: HelloCompiler.class to test a compiler
const outcls = "/clclcl" + Math.round(Math.random() * 100);
fs.mkdirSync(outcls, (error) => {});
const cmpUnit = "HelloCompiler.class";
const response2 = await fetch("http://localhost:8080/" + cmpUnit);
responseToReadable(response2).pipe(
  fs.createWriteStream(outcls + "/" + cmpUnit)
);
const response21 = await fetch("http://localhost:8080/module-info.class");
responseToReadable(response21).pipe(
  fs.createWriteStream(outcls + "/module-info.class")
);

let testdata = "/testdata" + Math.round(Math.random() * 100);
let testdata12 = "/testdata12";
console.log(testdata);
fs.mkdirSync(testdata, (error) => {});
fs.mkdirSync(testdata12, (error) => {});
const response3 = await fetch("http://localhost:8080/HelloCompiler2.java");
responseToReadable(response3).pipe(
  fs.createWriteStream(testdata12 + "/HelloCompiler2.java")
);

function pausecomp(millis) {
  var date = new Date();
  var curDate = null;
  do {
    curDate = new Date();
  } while (curDate - date < millis);
}

if (WebAssembly) {
  const go = new Go();
  let jacobinHome = "/jacobin" + Math.round(Math.random() * 100);
  // fs.mkdirSync(jacobinHome);
  // fs.mkdirSync(jacobinHome + '/jacobin_data');
  go.env = { JAVA_HOME: javaHome, HOME: jacobinHome };
  // go.argv = ['jacobin', '-version']
  go.argv = ["jacobin", outcls + "/" + cmpUnit]; //, '"' + javaHome + '/HelloCompiler2.java"']
  WebAssembly.instantiateStreaming(fetch("jacobin.wasm"), go.importObject).then(
    (result) => {
      let instance = result.instance;
      // instance.exports
      go.run(instance);
      pausecomp(5000);
      fs.readdir("/", (err, result) => {
        console.log(err);
        console.log(result);
        for (const resultElement of result) {
          fs.readdir("/" + resultElement, (err2, result2) => {
            console.log(err2);
            console.log(result2);
          });
        }
      });
      fs.readdir(jacobinHome, (err, result) => {
        console.log(err);
        console.log(result);
      });
    }
  );
} else {
  console.log("WebAssembly is not supported in your browser");
}

const linksList = [
  {
    title: "Docs",
    caption: "quasar.dev",
    icon: "school",
    link: "https://quasar.dev",
  },
  {
    title: "Github",
    caption: "github.com/quasarframework",
    icon: "code",
    link: "https://github.com/quasarframework",
  },
  {
    title: "Discord Chat Channel",
    caption: "chat.quasar.dev",
    icon: "chat",
    link: "https://chat.quasar.dev",
  },
  {
    title: "Forum",
    caption: "forum.quasar.dev",
    icon: "record_voice_over",
    link: "https://forum.quasar.dev",
  },
  {
    title: "Twitter",
    caption: "@quasarframework",
    icon: "rss_feed",
    link: "https://twitter.quasar.dev",
  },
  {
    title: "Facebook",
    caption: "@QuasarFramework",
    icon: "public",
    link: "https://facebook.quasar.dev",
  },
  {
    title: "Quasar Awesome",
    caption: "Community Quasar projects",
    icon: "favorite",
    link: "https://awesome.quasar.dev",
  },
];

export default defineComponent({
  name: "MainLayout",

  components: {
    EssentialLink,
  },

  setup() {
    const leftDrawerOpen = ref(false);

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
    };
  },
});
</script>
