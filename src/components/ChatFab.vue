<template>
  <!-- FAB Button -->
  <div class="louvorj-fab" :class="{ 'louvorj-fab--open': isOpen }">
    <!-- Backdrop (mobile) -->
    <div class="louvorj-fab__backdrop" @click="close"></div>

    <!-- Chat Panel -->
    <div class="louvorj-fab__panel" :class="{ 'louvorj-fab__panel--open': isOpen }">
      <!-- Panel Header -->
      <div class="lj-panel__header">
        <div class="lj-panel__avatar-wrap">
          <img :src="botAvatar" alt="LouvorJ.AI" class="lj-panel__avatar" />
          <span class="lj-panel__status-dot"></span>
        </div>
        <div class="lj-panel__info">
          <span class="lj-panel__name">{{ $t("chatbot.name") }}</span>
          <span class="lj-panel__status">{{ $t("chatbot.status") }}</span>
        </div>
        <div class="lj-panel__actions">
          <button class="lj-panel__btn" @click="clearChat" :title="$t('chatbot.new_conversation')">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
              <path d="M17.65 6.35A7.96 7.96 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/>
            </svg>
          </button>

        </div>
      </div>

      <!-- Messages -->
      <div ref="messagesArea" class="lj-panel__messages">
        <div class="lj-date" v-if="messages.length === 0 && !isTyping">
          {{ currentDate }}
        </div>

        <div
          v-for="(msg, index) in messages"
          :key="index"
          class="lj-msg"
          :class="{ 'lj-msg--user': msg.role === 'user' }"
        >
          <div class="lj-msg__avatar" :class="`lj-msg__avatar--${msg.role}`">
            <img v-if="msg.role === 'bot'" :src="botAvatar" alt="Bot" />
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="white">
              <path d="M12 4a4 4 0 0 1 4 4 4 4 0 0 1-4 4 4 4 0 0 1-4-4 4 4 0 0 1 4-4m0 10c4.42 0 8 1.79 8 4v2H4v-2c0-2.21 3.58-4 8-4z"/>
            </svg>
          </div>
          <div class="lj-msg__body">
            <div class="lj-bubble" :class="{ 'lj-bubble--user': msg.role === 'user' }" v-html="msg.text" />
            <div class="lj-bubble__meta">
              <span class="lj-bubble__time">{{ msg.time }}</span>
            </div>
            <div v-if="msg.sources && msg.sources.length" class="lj-sources">
              <div v-for="(s, si) in msg.sources" :key="si" class="lj-sources__item">
                <svg xmlns="http://www.w3.org/2000/svg" width="11" height="11" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M3.9 12c0-1.71 1.39-3.1 3.1-3.1h4V7H7a5 5 0 0 0-5 5 5 5 0 0 0 5 5h4v-1.9H7c-1.71 0-3.1-1.39-3.1-3.1M8 13h8v-2H8v2m9-6h-4v1.9h4c1.71 0 3.1 1.39 3.1 3.1s-1.39 3.1-3.1 3.1h-4V17h4a5 5 0 0 0 5-5 5 5 0 0 0-5-5z"/>
                </svg>
                <span>{{ s }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Typing indicator -->
        <div v-if="isTyping" class="lj-msg">
          <div class="lj-msg__avatar lj-msg__avatar--bot">
            <img :src="botAvatar" alt="Bot" />
          </div>
          <div class="lj-msg__body">
            <div class="lj-bubble lj-bubble--bot">
              <div class="lj-typing">
                <span></span><span></span><span></span>
              </div>
            </div>
          </div>
        </div>

        <!-- Quick Replies -->
        <div v-if="messages.length === 1 && !isTyping && showQuickReplies" class="lj-quick-replies">
          <button
            v-for="(qr, qi) in quickReplies"
            :key="qi"
            class="lj-quick-reply"
            @click="sendQuickReply(qr.text)"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="currentColor">
              <path d="M20 2H4c-1.1 0-2 .9-2 2v18l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2zm0 14H6l-2 2V4h16v12z"/>
            </svg>
            {{ qr.label }}
          </button>
        </div>
      </div>

      <!-- Input -->
      <div class="lj-panel__input">
        <input type="file" ref="fileInput" accept=".txt,.md,.pdf,.pptx,.ja" style="display:none" @change="handleFileUpload" />
        <button class="lj-attach-btn" @click="$refs.fileInput.click()" :title="$t('chatbot.send_file')">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="lj-attach-icon" style="color: rgba(0,0,0,0.4)">
            <path d="M21.44 11.05l-9.19 9.19a6 6 0 0 1-8.49-8.49l9.19-9.19a4 4 0 0 1 5.66 5.66l-9.2 9.19a2 2 0 0 1-2.83-2.83l8.49-8.48"/>
          </svg>
        </button>
        <textarea
          ref="inputField"
          v-model="inputText"
          class="lj-input"
          :placeholder="$t('chatbot.placeholder')"
          rows="1"
          @keydown.enter.exact.prevent="sendMessage"
          @input="autoResize"
        />
        <button
          class="lj-send"
          :class="{ 'lj-send--active': inputText.trim() }"
          :disabled="!inputText.trim() || isTyping"
          @click="sendMessage"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="currentColor">
            <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
          </svg>
        </button>
      </div>
    </div>

    <!-- FAB Trigger -->
    <button class="louvorj-fab__trigger" @click="toggle" :aria-label="isOpen ? $t('chatbot.close') : $t('chatbot.open')">
      <img :src="botAvatar" alt="LouvorJ.AI" class="louvorj-fab__icon" />
      <svg
        v-if="isOpen"
        class="louvorj-fab__close-icon"
        xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="white"
      >
        <path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
      </svg>
      <!-- Notification badge -->
      <span v-if="!isOpen && unreadCount > 0" class="louvorj-fab__badge">{{ unreadCount }}</span>
    </button>
  </div>
</template>

<script>
import Api from "@/services/Api.js";
import { buildSystemPrompt } from "@/utils/chatbot-rag";

const KNOWLEDGE = [
  {
    keywords: ["atalho", "tecla", "ctrl", "f1", "shortcut"],
    text: "Principais atalhos do LouvorJA:<br>&#8226; <strong>Ctrl+B</strong> &#8212; Busca r&aacute;pida<br>&#8226; <strong>Ctrl+L</strong> &#8212; Letras<br>&#8226; <strong>Ctrl+Enter</strong> &#8212; Reproduzir/Parar<br>&#8226; <strong>Ctrl+T</strong> &#8212; Transpor tom<br>&#8226; <strong>F1</strong> &#8212; Ajuda<br>&#8226; <strong>ESC</strong> &#8212; Fechar janelas<br>&#8226; <strong>Ctrl+P</strong> &#8212; Projetar<br>&#8226; <strong>Ctrl+G</strong> &#8212; Busca b&iacute;blica<br>&#8226; <strong>Ctrl+S</strong> &#8212; Salvar playlist<br>Use <strong>*</strong> na busca para qualquer trecho entre palavras.",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["transmit", "obs", "vmix", "stream", "projetar", "segundo monitor"],
    text: "Para transmitir no OBS/VMIX:<br>1. Menu &rarr; <strong>Transmitir</strong><br>2. Configure IP e porta<br>3. Clique <strong>Iniciar Servidor</strong><br>4. Copie a URL e insira como <strong>Navegador</strong> no OBS/VMIX<br>5. Customize o <strong>CSS</strong> para formata&ccedil;&atilde;o<br><br>Dica: se der erro, tente mudar a porta.",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["letra", "cifra", "acorde"],
    text: "O LouvorJA exibe letras e cifras em tempo real! Use <strong>Ctrl+L</strong> para abrir as letras. Na configura&ccedil;&atilde;o, ajuste tamanho da fonte, cores e formato. As cifras s&atilde;o transpostas automaticamente ao mudar o tom.",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["cole\u00e7\u00e3o", "coletanea", "playlist"],
    text: "Tipos de cole&ccedil;&otilde;es no LouvorJA:<br>&#8226; <strong>On-line</strong> &#8212; reproduz direto do YouTube<br>&#8226; <strong>Personalizadas</strong> &#8212; suas pr&oacute;prias playlists<br>&#8226; <strong>JA</strong> &#8212; Louvores dos Jovens Adventistas<br>&#8226; <strong>Min. M&uacute;sica</strong> &#8212; Minist&eacute;rio de M&uacute;sica<br><br>Crie, edite e exporte suas cole&ccedil;&otilde;es!",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["bibl", "vers\u00edc", "versic", "passage"],
    text: "Busca b&iacute;blica integrada! Pressione <strong>Ctrl+G</strong> para abrir. Suporta m&uacute;ltiplas vers&otilde;es b&iacute;blicas, busca por palavra-chave e navega&ccedil;&atilde;o por livro/cap&iacute;tulo/vers&iacute;culo.",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["liturg", "culto", "programa\u00e7\u00e3o", "programacao", "agenda", "escala"],
    text: "O m&oacute;dulo de Liturgia organiza a programa&ccedil;&atilde;o do culto:<br>&#8226; Crie itens agendados com datas<br>&#8226; Organize a sequ&ecirc;ncia do culto<br>&#8226; Vincule m&uacute;sicas e leituras b&iacute;blicas<br>&#8226; Exporte para slides de proje&ccedil;&atilde;o",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["css", "estilo", "formata\u00e7\u00e3o", "formatar", "apar\u00eancia", "fonte"],
    text: "Customize a apar&ecirc;ncia da proje&ccedil;&atilde;o e transmiss&atilde;o com <strong>CSS</strong>:<br>&#8226; Edite cores, fontes e tamanhos<br>&#8226; Formate a exibi&ccedil;&atilde;o das letras<br>&#8226; Personalize o fundo e layout<br>O CSS &eacute; aplicado em tempo real.",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["editor", "slide", "apresenta\u00e7\u00e3o"],
    text: "O <strong>Editor de Slides</strong> permite criar slides personalizados, gravar tempos e intervalos, formatar textos e imagens, e exportar para o m&oacute;dulo de proje&ccedil;&atilde;o. Ideal para cultos e eventos especiais!",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["provai", "vede", "v\u00eddeo", "video"],
    text: "O m&oacute;dulo <strong>Provai e Vede</strong> integra v&iacute;deos ao culto. Baixe os v&iacute;deos manualmente, cadastre na tela <strong>Itens Agendados</strong>, vincule &agrave; data do s&aacute;bado e adicione na Liturgia. O app N&Atilde;O possui os v&iacute;deos nativamente.",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["configurar", "configura\u00e7\u00e3o", "ajuste", "prefer\u00eancia"],
    text: "Configura&ccedil;&otilde;es do LouvorJA:<br>&#8226; <strong>Tema</strong> &#8212; claro ou escuro<br>&#8226; <strong>Fonte</strong> &#8212; tamanho e estilo<br>&#8226; <strong>Transposi&ccedil;&atilde;o padr&atilde;o</strong> &#8212; tom padr&atilde;o<br>&#8226; <strong>Ordem das músicas</strong> &#8212; como s&atilde;o listadas<br>&#8226; <strong>Idioma</strong> &#8212; portugu&ecirc;s e espanhol",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["export", "salvar", "arquivo", "slja", "mp3"],
    text: "Formatos de exporta&ccedil;&atilde;o do LouvorJA:<br>&#8226; <strong>.slja</strong> &#8212; formato nativo<br>&#8226; <strong>PDF</strong> &#8212; para impress&atilde;o<br>&#8226; <strong>MP3</strong> &#8212; &aacute;udio cantado e playback<br>Exporte m&u00fasicas, slides e playlists facilmente!",
    sources: ["louvorja.com/ajuda"],
  },
  {
    keywords: ["download", "baixar", "instalar", "mobile", "celular", "android", "windows"],
    text: "O LouvorJA est&aacute; dispon&iacute;vel para:<br>&#8226; <strong>Android</strong> &#8212; gratuito na Play Store<br>&#8226; <strong>Windows</strong> &#8212; download no site oficial<br>&#8226; <strong>Web</strong> &#8212; acesso pelo navegador<br><br>Acesse <strong>louvorja.com/download</strong>!",
    sources: ["louvorja.com/download"],
  },
];

const QUICK_REPLIES = [
  { label: "Buscar música", text: "Quero buscar uma música" },
  { label: "Categorias", text: "Quais categorias est\u00e3o dispon\u00edveis?" },
  { label: "Hin\u00e1rio", text: "Buscar no hin\u00e1rio" },
  { label: "Atalhos", text: "Quais s\u00e3o os atalhos do LouvorJA?" },
  { label: "Transmitir", text: "Como transmitir para OBS/VMIX?" },
  { label: "Download", text: "Como baixar o app?" },
];


export default {
  name: "ChatFab",
  data: () => ({
    botAvatar: new URL("@/assets/imgs/chatbot-avatar.jpg", import.meta.url).href,
    isOpen: false,
    messages: [],
    inputText: "",
    isTyping: false,
    welcomeShown: false,
    unreadCount: 0,
    showQuickReplies: false,
    quickReplies: QUICK_REPLIES,
    musicIndex: null,
    categories: null,
    hymnalData: null,
    musicLoaded: false,
    uploadingFile: false,
  }),
  computed: {
    currentDate() {
      return new Date().toLocaleDateString("pt-BR", { day: "numeric", month: "long" });
    },
  },
  methods: {
    getCurrentTime() {
      return new Date().toLocaleTimeString("pt-BR", { hour: "2-digit", minute: "2-digit" });
    },
    toggle() { this.isOpen ? this.close() : this.open(); },
    open() {
      this.isOpen = true;
      this.unreadCount = 0;
      this.$nextTick(() => {
        this.showWelcome();
        this.scrollToBottom();
        if (this.$refs.inputField) this.$refs.inputField.focus();
      });
      if (!this.musicLoaded) { this.musicLoaded = true; this.fetchMusicIndex(); }
    },
    close() { this.isOpen = false; this.showQuickReplies = false; },
    async sendMessage() {
      const text = this.inputText.trim();
      if (!text || this.isTyping) return;
      this.messages.push({ role: "user", text: this.escapeHtml(text), time: this.getCurrentTime() });
      this.showQuickReplies = false;
      this.inputText = "";
      this.$nextTick(() => {
        this.scrollToBottom();
        if (this.$refs.inputField) this.$refs.inputField.style.height = "auto";
      });
      this.isTyping = true;
      this.$nextTick(() => this.scrollToBottom());
      try {
        const resp = await this.generateBotResponse(text);
        this.isTyping = false;
        this.messages.push({ role: "bot", text: resp.text, time: this.getCurrentTime(), sources: resp.sources || [] });
        if (!this.isOpen) this.unreadCount++;
        this.$nextTick(() => {
          this.scrollToBottom();
          if (this.messages.length <= 2) this.showQuickReplies = true;
        });
      } catch (e) {
        this.isTyping = false;
        this.messages.push({ role: "bot", text: this.$t("chatbot.error"), time: this.getCurrentTime() });
        this.scrollToBottom();
      }
    },
    sendQuickReply(text) { this.inputText = text; this.sendMessage(); },
    detectIntent(text) {
      const t = text.toLowerCase();
      const isSearch = t.includes("buscar") || t.includes("procurar") || t.includes("achar") || t.includes("encontrar");
      const isMusic = t.includes("música") || t.includes("hino") || t.includes("som") || t.includes("louvor");
      if (isSearch && isMusic) return "music_search";
      if (t.includes("hinari") || t.includes("hino adventista") || /\d/.test(t)) return "hymnal_search";
      if (t.includes("categor") || t.includes("coleção") || t.includes("tipo") || t.includes("tema")) return "categories";
      return "knowledge";
    },
    async generateBotResponse(userText) {
      const intent = this.detectIntent(userText);
      switch (intent) {
        case "music_search": return await this.handleMusicSearch(userText);
        case "hymnal_search": return await this.handleHymnalSearch(userText);
        case "categories": return await this.handleCategories();
        case "knowledge": {
          const local = this.handleKnowledge(userText);
          if (local) return local;
          return await this.callLLM(userText);
        }
        default:
          return {
            text: this.$t("chatbot.guardrail"),
            sources: [],
          };
      }
    },
    fetchMusicIndex() {
      return new Promise((resolve) => {
        Api.get("json_db/pt_musics", null, (ok, data) => {
          if (ok) { this.musicIndex = data; resolve(data); }
          else { console.warn("[ChatFab] Failed to load music index"); resolve(null); }
        });
      });
    },
    fetchCategories() {
      if (this.categories) return Promise.resolve(this.categories);
      return new Promise((resolve) => {
        Api.get("pt/categories", null, (ok, data) => {
          if (ok) { this.categories = data; resolve(data); }
          else { console.warn("[ChatFab] Failed to load categories"); resolve(null); }
        });
      });
    },
    fetchHymnal() {
      if (this.hymnalData) return Promise.resolve(this.hymnalData);
      return new Promise((resolve) => {
        Api.get("pt/hymnal", { limit: 200 }, (ok, data) => {
          if (ok) { this.hymnalData = data; resolve(data); }
          else { console.warn("[ChatFab] Failed to load hymnal"); resolve(null); }
        });
      });
    },
    async handleMusicSearch(query) {
      await this.fetchMusicIndex();
      const arr = Array.isArray(this.musicIndex) ? this.musicIndex : (this.musicIndex?.data || []);
      if (!arr.length) return { text: "N\u00e3o consegui carregar o \u00edndice de músicas. Tente novamente.", sources: [] };
      const q = query.toLowerCase().replace(/quero|gostaria de|buscar|procurar|hino|som|louvor|sobre/gi, "").trim();
      if (!q) return { text: "Digite o nome ou parte do nome da música que deseja buscar.", sources: [] };
      const results = arr.filter(m => {
        const title = (m.title || m.name || "").toLowerCase();
        const artist = (m.artist || m.author || "").toLowerCase();
        return title.includes(q) || artist.includes(q);
      }).slice(0, 8);
      if (!results.length) return { text: `N\u00e3o encontrei resultados para "<strong>${this.escapeHtml(q)}</strong>". Tente outro termo!`, sources: [] };
      const html = results.map(m => {
        const name = m.title || m.name || "Sem t\u00edtulo";
        const parts = [m.hymnal, m.tone ? `Tom ${m.tone}` : null, m.artist || m.author || null].filter(Boolean);
        const info = parts.join(" \u2022 ") || "\u2014";
        return `<div class="lj-search-item"><div class="lj-search-item__name">${this.escapeHtml(name)}</div><div class="lj-search-item__info">${this.escapeHtml(info)}</div></div>`;
      }).join("");
      return { text: `Encontrei <strong>${results.length}</strong> resultado(s):<div class="lj-search-results">${html}</div>`, sources: ["LouvorJA M\u00fasicas"] };
    },
    async handleHymnalSearch(query) {
      const data = await this.fetchHymnal();
      const arr = Array.isArray(data) ? data : (data?.data || []);
      if (!arr.length) return { text: "N\u00e3o consegui carregar o hin\u00e1rio. Tente novamente.", sources: [] };
      const q = query.toLowerCase().replace(/buscar|hin[aá]rio|hino|adventista|n[úu]mero|numero|no|na/g, "").trim();
      const num = parseInt(q);
      let results;
      if (!isNaN(num) && num > 0) {
        results = arr.filter(h => String(h.number || h.num || h.numero || "") === String(num));
      } else if (q) {
        results = arr.filter(h => (h.title || h.name || "").toLowerCase().includes(q));
      } else {
        results = arr.slice(0, 10);
      }
      if (!results.length) return { text: "N\u00e3o encontrei esse hino. Tente digitar o n\u00famero ou nome!", sources: [] };
      const html = results.map(h => {
        const name = h.title || h.name || "Sem t\u00edtulo";
        const number = h.number || h.num || h.numero || "";
        const parts = [number ? `Hino ${number}` : null, h.tone ? `Tom ${h.tone}` : null].filter(Boolean);
        const info = parts.join(" \u2022 ") || "\u2014";
        return `<div class="lj-search-item"><div class="lj-search-item__name">${this.escapeHtml(name)}</div><div class="lj-search-item__info">${this.escapeHtml(info)}</div></div>`;
      }).join("");
      return { text: `Hin\u00e1rio \u2014 <strong>${results.length}</strong> resultado(s):<div class="lj-search-results">${html}</div>`, sources: ["Hin\u00e1rio Adventista"] };
    },
    async handleCategories() {
      const data = await this.fetchCategories();
      const arr = Array.isArray(data) ? data : (data?.data || []);
      if (!arr.length) return { text: "N\u00e3o consegui carregar as categorias. Tente novamente.", sources: [] };
      const html = arr.map(c => {
        const name = c.name || c.title || c.category || "Sem nome";
        const count = c.count || c.total || c.music_count || "";
        const info = count ? `${count} músicas` : "\u2014";
        return `<div class="lj-search-item"><div class="lj-search-item__name">${this.escapeHtml(name)}</div><div class="lj-search-item__info">${this.escapeHtml(info)}</div></div>`;
      }).join("");
      return { text: `<strong>Categorias dispon\u00edveis:</strong><div class="lj-search-results">${html}</div>`, sources: ["LouvorJA Categorias"] };
    },
    handleKnowledge(userText) {
      const text = userText.toLowerCase();
      for (const k of KNOWLEDGE) {
        if (k.keywords.some(kw => text.includes(kw))) return { text: k.text, sources: k.sources || [] };
      }
      return null; // no local match — will fall through to LLM
    },
    markdownToHtml(text) {
      if (!text) return "";
      return text
        .replace(/\*\*([^*]+)\*\*/g, "<strong>$1</strong>")
        .replace(/\*([^*]+)\*/g, "<em>$1</em>")
        .replace(/^### (.+)$/gm, "<h4>$1</h4>")
        .replace(/^## (.+)$/gm, "<h3>$1</h3>")
        .replace(/^# (.+)$/gm, "<h2>$1</h2>")
        .replace(/^\- (.+)$/gm, "<li>$1</li>")
        .replace(/^\d+\. (.+)$/gm, "<li>$1</li>")
        .replace(/\n{2,}/g, "<br><br>")
        .replace(/\n/g, "<br>");
    },

    async callLLM(userText) {
      const GROQ_API_URL = "https://api.groq.com/openai/v1/chat/completions";
      const GROQ_API_KEY = import.meta.env.VITE_GROQ_API_KEY;
      const MODEL = "llama-3.3-70b-versatile";
      const locale = this.$i18n?.locale || "pt";
      const lang = locale === "es" ? "español" : "português brasileiro";
      const systemPrompt = buildSystemPrompt(userText, lang);
      try {
        const res = await fetch(GROQ_API_URL, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Authorization": `Bearer ${GROQ_API_KEY}`,
          },
          body: JSON.stringify({
            model: MODEL,
            messages: [
              { role: "system", content: systemPrompt },
              ...this.messages.slice(-10).map(m => ({ role: m.role === "bot" ? "assistant" : "user", content: m.text.replace(/<[^>]*>/g, "") })),
              { role: "user", content: userText },
            ],
            max_tokens: 600,
            temperature: 0.6,
          }),
        });
        if (!res.ok) throw new Error(`Groq API ${res.status}`);
        const data = await res.json();
        const reply = data.choices?.[0]?.message?.content;
        return { text: reply || this.$t("chatbot.error"), sources: ["Groq LLM"] };
      } catch (e) {
        console.warn("[ChatFab] LLM call failed:", e);
        return {
          text: this.$t("chatbot.fallback"),
          sources: ["louvorja.com/ajuda"],
        };
      }
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;
      const maxBytes = 500 * 1024;
      if (file.size > maxBytes) {
        this.messages.push({ role: "bot", text: `O arquivo "${file.name}" excede o limite de 500 KB. Envie um texto mais curto ou copie e cole o conteudo aqui.`, time: this.getCurrentTime() });
        this.scrollToBottom();
        event.target.value = "";
        return;
      }
      const ext = file.name.split(".").pop().toLowerCase();
      const readable = ["txt", "md", "ja"];
      if (!readable.includes(ext)) {
        this.messages.push({ role: "user", text: `[Arquivo: ${file.name}]`, time: this.getCurrentTime() });
        this.showQuickReplies = false;
        this.$nextTick(() => this.scrollToBottom());
        this.isTyping = true;
        this.$nextTick(() => this.scrollToBottom());
        setTimeout(() => {
          this.isTyping = false;
          const helpText = `Recebi o arquivo "<strong>${file.name}</strong>". Para obter sugestoes de hinos baseadas no conteudo:<br><br>1. Abra o arquivo no seu computador<br>2. Copie o texto principal (tema do sermao, programa, etc.)<br>3. Cole aqui no chat<br><br>Assim posso analisar e sugerir hinos adequados para sua programacao!`;
          this.messages.push({ role: "bot", text: helpText, time: this.getCurrentTime() });
          this.scrollToBottom();
        }, 800);
        event.target.value = "";
        return;
      }
      this.uploadingFile = true;
      this.messages.push({ role: "user", text: `[Analisando: ${file.name}...]`, time: this.getCurrentTime() });
      this.showQuickReplies = false;
      this.$nextTick(() => this.scrollToBottom());
      this.isTyping = true;
      this.$nextTick(() => this.scrollToBottom());
      const reader = new FileReader();
      reader.onload = async (e) => {
        const content = e.target.result;
        const truncated = content.length > 3000 ? content.substring(0, 3000) + "\n...[conteudo truncado]" : content;
        this.uploadingFile = false;
        this.messages[this.messages.length - 1] = { role: "user", text: this.escapeHtml(`[Arquivo: ${file.name}]`), time: this.getCurrentTime() };
        this.scrollToBottom();
        try {
          const resp = await this.generateBotResponse(`Analise este conteudo de "${file.name}" e sugira hinos do LouvorJA relacionados:\n${truncated}`);
          this.isTyping = false;
          this.messages.push({ role: "bot", text: resp.text, time: this.getCurrentTime(), sources: resp.sources || [] });
          this.$nextTick(() => this.scrollToBottom());
        } catch (err) {
          this.isTyping = false;
          this.messages.push({ role: "bot", text: "Erro ao analisar o arquivo. Tente novamente ou copie o conteudo como mensagem.", time: this.getCurrentTime() });
          this.scrollToBottom();
        }
      };
      reader.onerror = () => {
        this.uploadingFile = false;
        this.isTyping = false;
        this.messages[this.messages.length - 1] = { role: "user", text: this.escapeHtml(file.name), time: this.getCurrentTime() };
        this.messages.push({ role: "bot", text: "Nao foi possivel ler o arquivo. Tente novamente ou copie e cole o conteudo.", time: this.getCurrentTime() });
        this.scrollToBottom();
      };
      reader.readAsText(file);
      event.target.value = "";
    },
    escapeHtml(text) {
      const d = document.createElement("div");
      d.textContent = text;
      return d.innerHTML;
    },
    autoResize() {
      const el = this.$refs.inputField;
      if (el) { el.style.height = "auto"; el.style.height = Math.min(el.scrollHeight, 100) + "px"; }
    },
    scrollToBottom() {
      const a = this.$refs.messagesArea;
      if (a) a.scrollTop = a.scrollHeight;
    },
    clearChat() {
      this.messages = [];
      this.welcomeShown = false;
      this.showQuickReplies = false;
      this.$nextTick(() => this.showWelcome());
    },
    showWelcome() {
      if (this.welcomeShown) return;
      this.welcomeShown = true;
      this.messages.push({
        role: "bot",
        text: this.$t("chatbot.welcome"),
        time: this.getCurrentTime(),
      });
    },
  },
};
</script>

<style scoped>
/* ========== FAB CONTAINER ========== */
.louvorj-fab {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
  font-family: 'Segoe UI', Roboto, -apple-system, sans-serif;
}

/* ========== FAB TRIGGER ========== */
.louvorj-fab__trigger {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 3px solid #2E86C1;
  background: linear-gradient(135deg, #2E86C1, #F7DC6F);
  box-shadow: 0 4px 16px rgba(46, 134, 193, 0.4), 0 2px 8px rgba(0, 0, 0, 0.15);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1), box-shadow 0.3s ease;
  position: relative;
  z-index: 2;
  padding: 0;
  overflow: hidden;
}
.louvorj-fab__trigger:hover {
  transform: scale(1.08);
  box-shadow: 0 6px 24px rgba(46, 134, 193, 0.5), 0 4px 12px rgba(0, 0, 0, 0.2);
}
.louvorj-fab__trigger:active { transform: scale(0.95); }

.louvorj-fab__icon {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
  transition: transform 0.3s ease;
}
.louvorj-fab--open .louvorj-fab__icon { transform: scale(0); position: absolute; }

.louvorj-fab__close-icon {
  position: absolute;
  transition: transform 0.3s ease;
}
.louvorj-fab:not(.louvorj-fab--open) .louvorj-fab__close-icon { transform: scale(0) rotate(-90deg); }
.louvorj-fab--open .louvorj-fab__close-icon { transform: scale(1) rotate(0deg); }

/* Badge */
.louvorj-fab__badge {
  position: absolute;
  top: -2px;
  right: -2px;
  width: 20px;
  height: 20px;
  background: #E74C3C;
  color: white;
  font-size: 11px;
  font-weight: 700;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid white;
  animation: badgePop 0.3s ease;
}

/* ========== BACKDROP (mobile) ========== */
.louvorj-fab__backdrop {
  display: none;
}

/* ========== CHAT PANEL ========== */
.louvorj-fab__panel {
  position: absolute;
  bottom: 72px;
  right: 0;
  width: 380px;
  height: 520px;
  background: #fff;
  border-radius: 20px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 8px 40px rgba(0, 0, 0, 0.18), 0 2px 12px rgba(0, 0, 0, 0.08);
  transform: scale(0.8) translateY(20px);
  transform-origin: bottom right;
  opacity: 0;
  visibility: hidden;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.3s ease, visibility 0.3s ease;
}
.louvorj-fab__panel--open {
  transform: scale(1) translateY(0);
  opacity: 1;
  visibility: visible;
}

/* ========== PANEL HEADER ========== */
.lj-panel__header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 16px;
  background: linear-gradient(135deg, #2E86C1, #1A6DA3);
  color: white;
}
.lj-panel__avatar-wrap {
  position: relative;
  flex-shrink: 0;
}
.lj-panel__avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid rgba(255,255,255,0.3);
}
.lj-panel__status-dot {
  position: absolute;
  bottom: 1px;
  right: 1px;
  width: 10px;
  height: 10px;
  background: #2ECC71;
  border-radius: 50%;
  border: 2px solid #2E86C1;
}
.lj-panel__info { flex: 1; display: flex; flex-direction: column; min-width: 0; }
.lj-panel__name { font-size: 14px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.lj-panel__status { font-size: 11px; opacity: 0.8; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.lj-panel__actions { display: flex; gap: 4px; flex-shrink: 0; }
.lj-panel__btn {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: none;
  background: rgba(255,255,255,0.15);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}
.lj-panel__btn:hover { background: rgba(255,255,255,0.25); }
.lj-panel__btn--close:hover { background: rgba(255,255,255,0.35); }

/* ========== MESSAGES AREA ========== */
.lj-panel__messages {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  background: #f8f9fa;
}
.lj-panel__messages::-webkit-scrollbar { width: 4px; }
.lj-panel__messages::-webkit-scrollbar-track { background: transparent; }
.lj-panel__messages::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.12); border-radius: 4px; }

.lj-date {
  text-align: center;
  font-size: 11px;
  color: #999;
  padding: 6px 0;
}

/* ========== MESSAGE ========== */
.lj-msg {
  display: flex;
  gap: 8px;
  align-items: flex-end;
  max-width: 85%;
}
.lj-msg--user {
  align-self: flex-end;
  flex-direction: row-reverse;
}

.lj-msg__avatar {
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  overflow: hidden;
}
.lj-msg__avatar--bot { width: 32px; height: 32px; min-width: 32px; background: #eee; }
.lj-msg__avatar--bot img { width: 100%; height: 100%; object-fit: cover; }
.lj-msg__avatar--user { width: 28px; height: 28px; min-width: 28px; background: #2E86C1; }

.lj-msg__body {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

/* ========== BUBBLE ========== */
.lj-bubble {
  padding: 10px 14px;
  font-size: 13.5px;
  line-height: 1.55;
  word-wrap: break-word;
  overflow-wrap: break-word;
  color: #333;
}
.lj-bubble--bot {
  background: #fff;
  border-radius: 16px 16px 16px 4px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.06);
}
.lj-bubble--user {
  background: linear-gradient(135deg, #2E86C1, #1A6DA3);
  color: white;
  border-radius: 16px 16px 4px 16px;
}

.lj-bubble__meta { padding: 2px 4px; }
.lj-bubble__time { font-size: 10px; color: #aaa; }
.lj-msg--user .lj-bubble__time { color: rgba(255,255,255,0.6); }

/* ========== SOURCES ========== */
.lj-sources {
  border-top: 1px solid #eee;
  margin-top: 2px;
  padding-top: 4px;
}
.lj-sources__item {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 11px;
  color: #2E86C1;
  padding: 1px 0;
  cursor: pointer;
}
.lj-sources__item:hover { text-decoration: underline; }

/* ========== TYPING ========== */
.lj-typing {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 0;
}
.lj-typing span {
  width: 7px;
  height: 7px;
  background: #bbb;
  border-radius: 50%;
  animation: typingBounce 1.4s infinite ease-in-out;
}
.lj-typing span:nth-child(2) { animation-delay: 0.2s; }
.lj-typing span:nth-child(3) { animation-delay: 0.4s; }
@keyframes typingBounce {
  0%, 60%, 100% { transform: translateY(0); opacity: 0.4; }
  30% { transform: translateY(-5px); opacity: 1; }
}

/* ========== QUICK REPLIES ========== */
.lj-quick-replies {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  padding: 4px 0;
}
.lj-quick-reply {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 14px;
  border: 1.5px solid #2E86C1;
  border-radius: 20px;
  background: transparent;
  color: #2E86C1;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}
.lj-quick-reply:hover {
  background: #2E86C1;
  color: white;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(46, 134, 193, 0.3);
}

/* ========== INPUT ========== */
.lj-panel__input {
  display: flex;
  align-items: flex-end;
  gap: 8px;
  padding: 12px 14px;
  background: #fff;
  border-top: 1px solid #eee;
}
.lj-input {
  flex: 1;
  background: #f0f2f5;
  border: 1.5px solid transparent;
  border-radius: 22px;
  padding: 10px 16px;
  resize: none;
  outline: none;
  font-family: inherit;
  font-size: 13.5px;
  color: #333;
  max-height: 100px;
  line-height: 1.4;
  transition: border-color 0.2s, background 0.2s;
}
.lj-input:focus { border-color: #2E86C1; background: #fff; }
.lj-input::placeholder { color: #aaa; }
.lj-send {
  width: 40px;
  height: 40px;
  min-width: 40px;
  border-radius: 50%;
  border: none;
  background: #e0e0e0;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}
.lj-send--active {
  background: linear-gradient(135deg, #2E86C1, #F7DC6F);
}
.lj-send:disabled { cursor: not-allowed; }

/* ========== ANIMATIONS ========== */
@keyframes badgePop {
  0% { transform: scale(0); }
  60% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

/* ========== MOBILE-FIRST RESPONSIVE ========== */
@media (max-width: 480px) {
  .louvorj-fab {
    bottom: 16px;
    right: 16px;
  }

  .louvorj-fab__trigger {
    width: 54px;
    height: 54px;
  }

  .louvorj-fab__backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.4);
    z-index: -1;
    opacity: 0;
    visibility: hidden;
    transition: opacity 0.3s ease, visibility 0.3s ease;
  }
  .louvorj-fab--open .louvorj-fab__backdrop {
    opacity: 1;
    visibility: visible;
  }

  .louvorj-fab__panel {
    position: fixed;
    bottom: 0;
    right: 0;
    left: 0;
    width: 100%;
    height: 100%;
    max-height: 100%;
    border-radius: 20px 20px 0 0;
    transform: translateY(100%);
    transform-origin: bottom center;
  }
  .louvorj-fab__panel--open {
    transform: translateY(0);
  }

  .lj-msg { max-width: 90%; }
}

@media (min-width: 481px) and (max-width: 768px) {
  .louvorj-fab__panel {
    width: 340px;
    height: 480px;
  }
}

/* Tablet/Desktop — default panel size applies */
@media (min-width: 769px) {
  .louvorj-fab__panel {
    width: 380px;
    height: 520px;
  }
}

/* ========== SEARCH RESULTS ========== */
.lj-attach-btn {
  background: none;
  border: none;
  padding: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: opacity 0.2s;
}
.lj-attach-btn:hover { opacity: 0.7; }
.lj-attach-icon { cursor: pointer; flex-shrink: 0; }

.lj-search-results {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 4px;
  max-height: 240px;
  overflow-y: auto;
}
.lj-search-item {
  padding: 8px 10px;
  background: rgba(0,0,0,0.03);
  border-radius: 8px;
  transition: background 0.2s;
}
.lj-search-item:hover {
  background: rgba(0,0,0,0.06);
}
.lj-search-item__name {
  font-size: 13px;
  font-weight: 600;
  color: #333;
}
.lj-search-item__info {
  font-size: 11px;
  color: #888;
  margin-top: 2px;
}
.lj-bubble--user + .lj-bubble__meta,
.lj-bubble--user ~ .lj-bubble__meta {
  display: flex;
  justify-content: flex-end;
}
</style>
