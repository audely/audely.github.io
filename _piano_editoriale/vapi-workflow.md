Perfetto, Antonio. Ecco una guida dettagliata per implementare un unico workflow n8n che gestisce i tre flussi principali dell’assistente vocale Vapi, integrato con il tuo calendario e con pianificazione appuntamenti reale.

⸻

🧠 Integrazione Vapi + Calendario + n8n

🔧 Guida completa per gestire:
	•	✅ Aggiornamento automatico della Knowledge Base
	•	✅ Verifica in tempo reale della disponibilità
	•	✅ Pianificazione appuntamenti al termine della chiamata

⸻

🎯 Obiettivo

Creare un singolo workflow n8n con 3 punti di ingresso distinti, ciascuno gestito da un Webhook node diverso, per:
	1.	update-availability: aggiornare ogni 12h la Knowledge Base di Vapi
	2.	check-availability: ricevere richieste assistant-request e rispondere dinamicamente
	3.	schedule-call: ricevere call.ended con i dati della conversazione, salvare o agire

⸻

🧩 PANORAMICA FLUSSI

Nome Webhook	Metodo	Attivazione	Cosa fa
/update-availability	POST	Ogni 12h via cron	Genera file .txt con slot liberi e lo carica su Vapi
/check-availability	POST	Evento assistant-request da Vapi	Analizza richiesta, controlla calendario, risponde
/schedule-call	POST	Evento call.ended da Vapi	Estrae dati, salva l’appuntamento, invia notifiche, ecc.


⸻

🔧 IMPOSTAZIONI SU VAPI

1. Imposta Server URL

Vai su Dashboard Vapi → Assistant → Settings → Server Settings e configura:

Campo	Valore
Server URL	https://TUO_DOMINIO/webhook/check-availability
Secret Token	(opzionale, usalo se vuoi validare le richieste)
Headers	(opzionali, ad esempio Authorization se vuoi firmare)

🔁 Vapi chiamerà check-availability ad ogni richiesta dell’utente all’agente.

⸻

2. Imposta eventi webhook per call.ended

Sempre in Server Settings, abilita:
	•	✅ call.ended tra gli eventi server
	•	Vapi invierà una richiesta a https://TUO_DOMINIO/webhook/schedule-call ogni volta che una chiamata termina

⸻

🏗️ STRUTTURA WORKFLOW n8n

🟡 1. Nodo Webhook: update-availability (manual trigger + cron)
	•	Metodo: POST
	•	URL: /webhook/update-availability
	•	Trigger secondario: Cron → ogni 12h
	•	Azioni:
	•	Google Calendar → Elenca eventi disponibili (es. titolo “Disponibile” o slot vuoti)
	•	Genera file .txt con i prossimi 3 slot in linguaggio naturale
	•	HTTP Request → API Vapi POST /files
	•	Headers: Authorization: Bearer <PRIVATE_API_KEY>
	•	Body: multipart/form-data con:
	•	file: contenuto testuale
	•	type: "knowledge"
	•	name: "disponibilita-settimanale"

⸻

🔵 2. Nodo Webhook: check-availability (assistant-request)
	•	Metodo: POST
	•	URL: /webhook/check-availability

Flusso:
	1.	Riceve payload con:

{
  "message": {
    "type": "assistant-request",
    "content": "Vorrei prenotare lunedì alle 10"
  }
}


	2.	Estrai data/ora dalla frase (ChatGPT, LangChain, Date & Time Parser)
	3.	Verifica disponibilità sul calendario
	4.	Rispondi con:

{
  "response": "Lunedì alle 10 non è disponibile. Ho disponibilità martedì alle 11 o mercoledì alle 18. Quale preferisci?"
}



⸻

🟢 3. Nodo Webhook: schedule-call (call.ended)
	•	Metodo: POST
	•	URL: /webhook/schedule-call

Flusso:
	1.	Riceve:

{
  "message": {
    "type": "call.ended",
    "call": {
      "transcript": "...",
      "startTime": "...",
      "endTime": "...",
      "assistant_id": "...",
      "phone": "+39...",
      "callId": "...",
      ...
    }
  }
}


	2.	Estrai dati da transcript o payload:
	•	Nome, email, orario scelto
	3.	Crea appuntamento nel calendario
	4.	Invia notifica (email, Telegram, CRM, database, ecc.)

⸻

📘 Prompt consigliato per l’agente Vapi

Ciao! Sono l’assistente vocale di Audely.
Antonio è disponibile per appuntamenti in orari limitati, che trovi nel documento “disponibilità settimanale”.

Posso proporti subito i prossimi slot liberi. Se preferisci un altro momento, chiedimelo pure e controllerò per te!


⸻

🔐 Sicurezza consigliata
	•	✅ Valida il Secret Token se configurato in Vapi (via header)
	•	✅ Aggiungi logging per debugging (es. transcript, errori calendario)
	•	✅ Usa Rate limit o Queue se temi troppe chiamate simultanee

⸻

🧪 Test suggeriti
	1.	Fai una chiamata e chiedi uno slot disponibile: → Vedi se risponde bene
	2.	Chiedi uno non disponibile: → Verifica che attivi check-availability
	3.	Completa una chiamata → Vedi se scatta schedule-call
	4.	Verifica aggiornamento automatico di update-availability da cron

⸻

🧩 Estensioni future possibili
	•	Sincronizzazione bidirezionale con Cal.com o Notion
	•	Salvataggio lead in un CRM (es. HubSpot, Airtable, Supabase)
	•	Invio link Google Meet/Zoom automatico
	•	Dashboard con tutte le chiamate effettuate + estrazione analytics

⸻

✅ Checklist finale

Cosa	Stato
Assistant Vapi configurato con webhook	⬜
Knowledge Base creata e collegata	⬜
Workflow n8n con 3 webhook configurati	⬜
Connessione a Google Calendar testata	⬜
Testato end-to-end con chiamata reale	⬜


⸻

🔗 Vuoi che ti generi anche lo YAML completo del workflow n8n?

Posso crearti il file workflow.json da importare direttamente. Mi basta sapere:
	•	Il tuo fuso orario
	•	Quali orari vuoi includere nei suggerimenti automatici
	•	Se preferisci salvare gli appuntamenti anche su file o database

Fammi sapere e lo creo subito per te ✅

Hai centrato un punto cruciale per fare business con Vapi: il costo per ogni chiamata, anche per test o curiosità, può diventare un freno enorme all’adozione sul tuo sito o nei funnel di acquisizione.

Vediamo la situazione reale e poi come puoi risolverla strategicamente.

⸻

📉 Il problema
	•	Ogni chiamata con Vapi costa ~0,50$ / minuto, anche se l’utente non lascia dati o gioca con l’assistente.
	•	Quindi una demo pubblica aperta rischia di farti bruciare budget senza ROI, soprattutto se finisce su Reddit o TikTok…

⸻

🔍 Cosa offre Vapi ufficialmente

Al momento:

Opzione	Disponibilità gratuita
API Key gratuita	✅ (con limite)
Credito gratuito iniziale	✅ ($5 di credito per test)
Possibilità di “sandbox”	❌ (non esiste una modalità gratuita illimitata per test pubblici)

Non c’è una modalità “demo pubblica gratuita illimitata”, né per web né per telefono.

⸻

✅ Soluzioni pratiche per offrire una demo senza farti rovinare

1. 🎯 Limita le demo pubbliche solo al web (non via telefono)
	•	Il widget web (via microfono browser) consuma meno risorse di una telefonata
	•	Puoi usare shorter prompt, quick timeout, no speech after 30s = end

2. 🛡️ Usa token di accesso o una landing riservata
	•	Mostra il widget solo a chi ha ricevuto un codice d’accesso
	•	Oppure metti la demo su una landing privata, visibile solo da chi la chiede

3. 🧠 Cambia comportamento in base al tipo di utente

Nel prompt iniziale, l’agente può distinguere:
	•	Se non riceve nome/email, chiude dopo 45 secondi
	•	Se riceve un interesse concreto, apre la scheduling

4. 🔁 Limita il numero di chiamate per IP o browser

Puoi farlo lato frontend con JS + localStorage o backend con una logica su n8n:

if (localStorage.getItem('vapiCallCount') >= 1) {
  alert("Hai già provato la demo. Vuoi prenotare una vera chiamata?");
}

5. 🚫 Disabilita temporaneamente il microfono per chi non interagisce

Con un minimo di JS puoi fare:
	•	Se l’utente non parla entro 10 secondi → chiudi la conversazione
	•	Se dice frasi tipo “fammi ridere”, “testo”, “scherzo” → fallback e chiusura

⸻

💡 Idea pro: Demo offline
	•	Registra uno screencast reale di 60 secondi con l’assistente in azione
	•	Lo metti in autoplay in homepage con call-to-action “Prova ora”
	•	Solo se clicca → attivi la vera sessione Vapi

⸻

🔁 Alternativa per il business: crea una “sandbox temporanea”

Puoi creare una Vapi Demo API Proxy su n8n che:
	1.	Riceve tutte le richieste Vapi
	2.	Le inoltra all’agente vero solo se:
	•	L’utente ha compilato nome/email
	•	È la prima chiamata
	•	Ha accettato i termini
	3.	Altrimenti gli risponde con:
“Ciao! Vuoi davvero vedere cosa può fare la mia assistente? Prenota una demo gratuita oppure contattami!”

⸻

🚀 In sintesi: strategia consigliata

Livello	Azione
🌐 Web	Widget disponibile solo via web, timeout veloce
🎯 Target	Accesso limitato con codice, form o email
🧪 Demo	Usa screencast, video, funnel guidati
🛠️ n8n	Gestione intelligente delle chiamate per evitare sprechi
📈 Business	Offri demo vere solo se l’utente lascia un contatto verificabile


⸻

Se vuoi, ti preparo:
	•	Un mini script JS che controlla accessi demo
	•	Un workflow n8n che blocca chiamate di test o multiple
	•	Un video demo registrabile via Sora/OpenAI con voce sintetica dell’agente

Fammi sapere che strada vuoi seguire!


# chatbot classico

Ecco un chatbot AI completo per il tuo sito Jekyll, integrato in HTML e JavaScript, senza librerie esterne. È personalizzato per Audely, usa l’API OpenAI e mostra un’icona 💬 fissa in basso a destra.

✅ Cosa fare ora:
	1.	Sostituisci INSERISCI_LA_TUA_API_KEY_OPENAI con la tua chiave privata.
	2.	Incolla questo codice nel tuo layout (es. default.html) oppure in una singola pagina .md o .html.
	3.	Se vuoi risposte più economiche, cambia gpt-4o in gpt-3.5-turbo.

🔐 Sicurezza

Non esporre la tua chiave OpenAI in produzione!
→ Possiamo trasformare questo frontend in un sistema proxy via n8n o serverless con chiave nascosta lato backend.

⸻

🧠 Vuoi anche:
	•	una knowledge base Qdrant con le risposte automatiche alle domande frequenti?
	•	un flusso di prenotazione appuntamenti via Calendly o API?
	•	la trascrizione salvata?

Fammi sapere e ti espando il sistema.

Ottima domanda: ti spiego come proteggere la tua API Key OpenAI usando una funzione serverless (senza bisogno di server sempre attivi) o con n8n, così che il tuo chatbot funzioni in sicurezza anche sul sito pubblico.

⸻

🧰 Obiettivo

👉 Invece di esporre la chiave OpenAI nel JavaScript visibile ai visitatori, creiamo un intermediario (proxy) che:
	1.	Riceve il messaggio da utente (POST)
	2.	Chiama l’API OpenAI con la tua API key (nascosta)
	3.	Restituisce la risposta al frontend

⸻

🚀 Soluzioni per il backend sicuro (proxy API)

Soluzione	Vantaggi	Dove si ospita
🟢 n8n self-hosted	Hai già tutto pronto	Sul tuo Raspberry o server
🔵 Vercel / Netlify function	Gratis, scalabile	Serverless
🟠 Cloudflare Workers	Ultra veloce, gratuito	Perfetto per progetti piccoli
⚪ Express API su server	Più controllo	Ma serve hosting sempre attivo


⸻

✅ Guida: implementazione con n8n

Hai già n8n installato. Quindi:

1. Crea un nuovo Webhook trigger in n8n
	•	Metodo: POST
	•	Path: /chatbot-openai

2. Aggiungi un nodo HTTP Request (verso OpenAI)

POST https://api.openai.com/v1/chat/completions
Authorization: Bearer LA_TUA_API_KEY
Content-Type: application/json
Body:
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": "Sei un assistente virtuale italiano per Audely."
    },
    {
      "role": "user",
      "content": {{$json["message"]}}
    }
  ],
  "temperature": 0.6
}

3. Aggiungi un nodo Set per ritornare la risposta
	•	Estrai data.choices[0].message.content e lo rimandi al client

4. Attiva il webhook e copia l’URL

Es. https://n8n.audely.github.io/webhook/chatbot-openai

⸻

🧑‍💻 Ora modifica il JS nel tuo sito

const ENDPOINT = "https://n8n.audely.github.io/webhook/chatbot-openai";
// sostituisci fetch() con:
const res = await fetch(ENDPOINT, {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({ message: text })
});


⸻

🔒 Risultato

✅ La tua chiave OpenAI è protetta
✅ Il chatbot funziona senza esporre segreti
✅ Facile da scalare o estendere (es. salvataggio Qdrant, prenotazioni…)

⸻

Vuoi che ti prepari il workflow n8n già pronto da importare?

Posso farlo in JSON così lo carichi al volo. Ti basta dirmelo.
