# Peer Case Rounds: complete beginner's guide

**Peer Case Rounds** is a voice chatbot. You join a group of **four** medical trainees (you plus three simulated peers, for example **Amira**, **John** and **Priya**), with an **equal gender mix: two women and two men**, who discuss the **same infant patient** together, like real case rounds without a supervisor. The group works through:

**Case presentation → Symptoms and key findings → Differential diagnosis → Cause → Medical tests → Treatment → Final diagnosis → Prognosis → Wrap-up**

At the end you get a short **point-form evaluation** of your **preciseness**, **explanations** and **constructiveness**, with concrete tips to improve your discussion skills.

**What it does**
- Reads the **case file you choose** and quietly works out the final diagnosis, but the peers do not reveal it until the Final diagnosis phase.
- Adapts to **your level** (pre-clinical student up to fellow): the more experienced you are, the more advanced the questions and answers.
- Each peer has a **style** you can choose: Analytical and diagnostic, Jargon-dense and efficient, Empathetic and ethical, or Collaborative and egalitarian. The tone balances professional growth with peer camaraderie.
- Replies are **short**, and **no abbreviations** are used (every term is spelled out).
- **Equal gender mix:** you choose your gender, and the three peers are set automatically so the group is two women and two men. Each peer's **voice matches their gender** (female voices for women, male voices for men). Personalities and expertise are not tied to gender.
- The peers **speak aloud** in different voices and **listen** to your spoken answers.
- **⏸ Hold** pauses, **▶ Continuation** carries on, **↺ Reset** clears the case rounds, and saying **"stop"** ends the discussion and starts the evaluation.

---

## What's in the folder

| File | What it is |
|---|---|
| `index.html` | The chatbot page. This is your website. |
| `worker.js` | Code for your Cloudflare Worker. It contains **no** secrets. |
| `cases/cases.json` | The list of shared cases shown in the menu. |
| `cases/floppy-infant-6-weeks.txt` | Sample case: floppy infant (diagnosis withheld). |
| `cases/fever-and-rash-5-months.txt` | Sample case: fever and rash (diagnosis withheld). |
| `cases/case-template.txt` | Blank template for writing new cases. |
| `README.md` | This guide. |

## How the pieces fit together

```
Your browser  ──────►  Cloudflare Worker  ──────►  Claude
(GitHub Pages:          (worker.js: keeps          (does the
 index.html)             your API key secret)       thinking)
```

The website never sees your Anthropic key. Only the Worker has it, and the Worker only answers **your** GitHub website, and only when the right **access code** is given.

---

## Before you start: what you need

- A computer with **Google Chrome** or **Microsoft Edge** (best for voice; Safari and Firefox can type but voice input may not work).
- A **GitHub** account (free): https://github.com
- A **Cloudflare** account (free) with the Worker you made before, **Infant Case Rounds**: https://dash.cloudflare.com
- An **Anthropic Console** account with some credit: https://console.anthropic.com
- A microphone and speakers (or a headset, which works best: it stops the microphone hearing the peers).
- About **30 to 40 minutes** the first time.

Tip: keep this guide open in one window and the websites in another window.

---

## PART 0: Try it inside Claude first (no setup, 5 minutes)

You can test the chatbot straight away inside Claude, before doing any of the setup.

1. Open the **Peer Case Rounds** link that Claude gave you.
2. The badge at the top right says **"Running in Claude (test mode)"**.
3. **Step 1:** choose a shared case (for example "Floppy infant, 6 weeks old"), or open your own .txt or .docx case file.
4. **Step 2:** type your first name and choose your **gender** and level. The peers are balanced to two women and two men.
5. **Step 3:** keep or change your peers' names and styles. Click **🔊 try** in a peer card to hear that peer's voice.
6. Click **▶ Start case rounds**. If Claude asks for permission to answer or to speak, click **Allow**.
7. The peers talk. When it is **your turn**, either type and press **Enter**, or click **🎤**, speak, and pause (inside Claude you click 🎤 once for each turn).
8. Type or say **stop** to finish and see your evaluation.

Differences in test mode:
- PDF case files cannot be read (use .txt or .docx).
- Download and Print are hidden (use **📋 Copy report**).
- **Voices:** Step 4 has a **Voices** choice. Keep **Browser voices, labelled female or male**: these are matched to each peer's gender automatically. If you hear no voices with that setting, switch to **Claude's own voices**. Claude's four voices (airy, glassy, rounded, buttery) are not labelled by gender, so click **🔊 try** in each peer card and pick a voice that matches the peer's gender. When you are happy with it, put it on your own website with Parts 1 to 6 below.

---

## PART 1: Unzip the files (2 minutes)

1. Find **peer-rounds.zip** in your **Downloads** folder.
2. Unzip it:
   - **Windows:** right-click the file → **Extract All…** → **Extract**.
   - **Mac:** double-click the file.
3. You now have a folder called **peer-rounds**. Open it and check that you can see `index.html`, `worker.js`, `README.md` and a folder called `cases`.

---

## PART 2: Check your Anthropic account (3 minutes)

The Claude API is paid separately from any Claude subscription (Pro or Max plans do not include API credit).

1. Go to **https://console.anthropic.com** and log in.
2. In the left menu, click **Billing**. Check that you have credit (for example US $5 to $10). If not, click **Buy credits** and follow the steps.
3. Your Worker already has an API key, so you do **not** need a new one.
   *Only if you have lost it or are starting a new Worker:* left menu → **API Keys** → **Create Key** → give it a name (for example `peer-rounds`) → **Add** → **copy the key immediately** (it starts with `sk-ant-` and is shown only once). Paste it into Notepad for a moment. Never put it on GitHub.

---

## PART 3: Update your Cloudflare Worker (about 10 minutes)

You will reuse your existing Worker, **Infant Case Rounds**, and replace its code. Your saved variables (API key, access code and so on) are **kept** when you change the code. The new code is written to keep serving your **Doctor Bernd** and **Parent Conversation Simulator** chatbots as well.

> No Worker yet, or you deleted it? Follow the **Appendix: build a new Worker from scratch** at the end, then continue with Part 4.

### 3.1 Open the Worker
1. Go to **https://dash.cloudflare.com** and log in.
2. In the left menu, click **Compute (Workers)** → **Workers & Pages**. (On some accounts it is just called **Workers & Pages**.)
3. In the list, click your Worker, **infant-case-rounds**.

### 3.2 Back up the old code (safety step, do not skip)
1. Click **Edit code** (top right, `</>` icon). A code editor opens.
2. Click anywhere inside the code. Press **Ctrl+A** (Windows) or **Cmd+A** (Mac) to select everything, then **Ctrl+C** / **Cmd+C** to copy.
3. Open **Notepad** (Windows: Start menu → type "Notepad") or **TextEdit** (Mac: Applications → TextEdit). Paste with **Ctrl+V** / **Cmd+V**.
4. Save it on your desktop as `old-worker-backup.txt`.
   If anything ever goes wrong, you can paste this old code back.

### 3.3 Paste in the new code
1. On your computer, open `worker.js` from the **peer-rounds** folder in a plain text editor:
   - **Windows:** right-click `worker.js` → **Open with** → **Notepad**.
   - **Mac:** right-click → **Open With** → **TextEdit**.
2. Select everything (**Ctrl+A** / **Cmd+A**) and copy it (**Ctrl+C** / **Cmd+C**).
3. Go back to the Cloudflare code editor. Click inside the code, select everything (**Ctrl+A** / **Cmd+A**) and press **Delete**. The editor is now empty.
4. Paste (**Ctrl+V** / **Cmd+V**). The first lines should read `/** Cloudflare Worker: a safe Claude API proxy for GitHub Pages chatbots.`
5. Click the blue **Deploy** button (top right). Confirm if it asks.

### 3.4 Check the variables
1. Go back to the Worker's main page (click the Worker name at the top, or the back arrow).
2. Click **Settings** → **Variables and Secrets**.
3. Check that these three exist. If one is missing, click **+ Add**, choose the **Type**, type the **Name** exactly, paste the **Value**, and click **Save**.

| Name (type exactly) | Type | Value |
|---|---|---|
| `ANTHROPIC_API_KEY` | Secret | your key from console.anthropic.com |
| `ACCESS_CODE` | Secret | a passphrase you choose, for example `rounds-2026` |
| `ALLOWED_ORIGINS` | Text | `https://YOUR-GITHUB-USERNAME.github.io` |

**About ALLOWED_ORIGINS:**
- Write only `https://` + your GitHub username + `.github.io`. No repository name.
- ✅ `https://jsmith.github.io`  ❌ `https://jsmith.github.io/peer-rounds/`
- The name must end with an **S**.
- If it already contains your github.io address (from Doctor Bernd or the Parent Conversation Simulator), **leave it as it is**. All your repositories share that one address.

**Optional model variables (Text):** `MODEL_FAST` = `claude-haiku-5-5`, `MODEL_BALANCED` = `claude-sonnet-5-5`, `MODEL_THOROUGH` = `claude-opus-5-5`. These are already built in, so you can leave them out. If you set `MODEL_BALANCED = claude-sonnet-5-5` before, that is fine.

4. If you added or changed anything, click **Deploy**. **Variables only work after you deploy.**

### 3.5 Test the Worker
1. On the Worker's main page, find its address. It looks like `https://infant-case-rounds.YOUR-SUBDOMAIN.workers.dev`. **Copy it** and paste it into Notepad: you need it in Part 4.
2. Paste the address into a new browser tab and press **Enter**. You should see text like:
   `{"ok":true,"service":"Claude chatbot proxy (Peer Case Rounds, Doctor Bernd, Parent Conversation Simulator and other GitHub Pages chatbots) v3","apiKeySet":true,"accessCodeSet":true,...}`
   - `"apiKeySet":false` → add the `ANTHROPIC_API_KEY` secret (step 3.4) and Deploy.
   - `"allowedOrigins":"NOT SET..."` → add `ALLOWED_ORIGINS` (step 3.4) and Deploy.
   - The text does **not** mention **"Peer Case Rounds"** → the old code is still running. Repeat step 3.3 and click **Deploy**.

---

## PART 4: Put the chatbot on GitHub (about 10 minutes)

### 4.1 Create a new repository
A repository ("repo") is a project folder on GitHub.
1. Go to **https://github.com** and sign in. (No account? Click **Sign up** and follow the steps; choose a short username, because it becomes part of your web address.)
2. Click the **+** at the top right → **New repository**.
3. **Repository name:** `peer-rounds`
4. Choose **Public**. Free GitHub Pages needs a public repository. The page contains no secrets.
5. Leave **Add a README file** **unticked**.
6. Click the green **Create repository** button.

### 4.2 Upload the files
1. On the new, empty repository page, click the blue link **uploading an existing file**.
2. Open your unzipped **peer-rounds** folder on your computer.
3. Select **index.html**, **README.md**, **worker.js** and the **cases** folder (hold **Ctrl** on Windows or **Cmd** on Mac and click each one), then drag them all into the GitHub upload box.
   Dragging the `cases` folder keeps its files inside `cases/`, which is required.
4. Wait until all files appear in the list under the box (you should see `cases/cases.json` and the case files).
5. Scroll down and click the green **Commit changes** button.

### 4.3 Put your Worker address into the page (recommended)
Then learners only need to type the access code.
1. In your repository, click **index.html**.
2. Click the **pencil icon ✏️** (Edit this file), at the top right of the file.
3. Press **Ctrl+F** / **Cmd+F** and search for `WORKER_URL`. You will find:
   ```js
   WORKER_URL: "",
   ```
4. Click between the two quotation marks and paste your Worker address:
   ```js
   WORKER_URL: "https://infant-case-rounds.YOUR-SUBDOMAIN.workers.dev",
   ```
   Keep both quotation marks and the comma.
5. Click **Commit changes…** (top right) → **Commit changes**.

### 4.4 Switch on GitHub Pages
1. In the repository, click **Settings** (gear icon, top menu).
2. In the left menu, click **Pages**.
3. Under **Build and deployment** → **Source**, choose **Deploy from a branch**.
4. Under **Branch**, choose **main** and **/ (root)**, then click **Save**.
5. Wait 1 to 3 minutes, then refresh the page. A box appears: **"Your site is live at https://YOUR-USERNAME.github.io/peer-rounds/"**. Click **Visit site**.

---

## PART 5: First run (5 minutes)

1. Open `https://YOUR-USERNAME.github.io/peer-rounds/` in **Chrome** or **Edge**.
2. Click **⚙ Connection** (top right):
   - The Worker address should already be filled in. If not, paste it.
   - Type your **access code** (your `ACCESS_CODE` from Cloudflare).
   - Click **Test connection**. You should see **✔ Connected. Access code accepted.**
   - Click **Save and close**. Each browser only needs this once. The badge changes to **"Connected to your Worker"**.
3. **Step 1, Patient case file:** choose a shared case from the list, **or** click **Choose file** and pick a case from your computer (.txt, .md, .docx or .pdf), **or** drag the file onto the dashed box. Click **📄 Preview case file** to check it.
4. **Step 2, You:** type your first name, choose your **gender** and your **level of training**.
   - Your gender is used only to balance the group: if you are female, the peers are one woman and two men; if you are male, two women and one man. If you choose **Prefer not to say**, the peers are two women and one man.
5. **Step 3, Your peer group:** a green line confirms **"Equal gender mix: 2 women and 2 men"**. Keep or change the three peers' **names**, **gender**, **styles** and **voices**.
   - Each voice list shows voices of the peer's gender first. **Automatic** picks a different matching voice for each peer. Click **🔊 try** to hear it.
   - If you change a peer's gender, the name and voice change to match. If the mix is no longer 2 and 2, a warning appears with a **Make it 2 and 2** button. Choose their **seniority** (same as you, mixed, or more senior) and the **length** of the discussion. Keep all **discussion tones** ticked unless you want to leave one out.
6. **Step 4, Voice and quality:** keep **Peers speak aloud** and **Hands-free** ticked. Choose the accent and click **🔊 Test the peers' voices**. Leave quality on **Balanced**.
7. Click **▶ Start case rounds**. When the browser asks to use the microphone, click **Allow**.

---

## PART 6: During the case rounds

- The peers read the case (10 to 30 seconds), present it, greet you and ask you a question **aloud**.
- When they finish, the label says **Your turn** and the microphone button turns **red** (hands-free). Answer out loud. After your pause (2.5 seconds by default) the answer is sent. You can also click **🎤** to send at once, or **type** and press **Enter**.
- The bar at the top shows the phase, your number of turns and the time.
- **📄 Show the patient case file** lets you reread the case at any time.

| Button | What it does |
|---|---|
| **⏸ Hold** | Pauses everything: voices, microphone and timer. |
| **▶ Continuation** | Carries on where you paused (the peer repeats the line they were saying). |
| **↺ Reset** | Clears the case rounds. Choose **Restart this case** (same case and settings, from the beginning) or **Back to setup** (choose a new case or level). |
| **⏹ End and evaluate** | Ends now and goes to the evaluation. |
| Say or type **"stop"** | Same as End and evaluate. It must be said on its own (for example "stop" or "okay, stop"): "I would stop the feeds" does **not** end the session. |

**Tip:** wear a headset or keep the speaker volume moderate, so the microphone does not pick up the peers' voices.

---

## PART 7: The evaluation

After "stop" (or when the group has wrapped up the case), you get a short point-form summary:
- a one-sentence **overall impression**
- scores out of 5 for **preciseness**, **explanations** and **constructiveness**, each with one or two bullet points about what you actually said
- **final diagnosis**: what you said compared with the correct answer
- up to 3 **strengths**
- 2 to 4 **ways to improve your discussion skills**, each with one way to practise
- one **focus for next time**
- fold-out sections: the case answer key (teaching points) and the full transcript

Buttons: **🔊 Hear summary**, **📋 Copy report**, **⬇ Download report**, **🖨 Print or save as PDF**, **↺ Same case again**, **New case or level**.

---

## PART 8: Check your other chatbots still work

All your chatbots share one Worker. After updating it, open your **Doctor Bernd** page and your **Parent Conversation Simulator** page, and start a short session in each.
- **They work:** you are done. 🎉
- **One shows an error:** go back to Cloudflare → your Worker → **Edit code**, paste the code from `old-worker-backup.txt`, and click **Deploy**. Your old chatbots work again. Then give Peer Case Rounds its **own** Worker by following the Appendix (name it `peer-rounds`), and put that Worker's address in `index.html` (Part 4.3) and in ⚙ Connection. You can also tell Claude the exact error message and ask for one Worker that serves all three.

---

## Adding your own cases

**Just for you:** click **Choose file** each time.

**For everyone (the shared list):**
1. In your GitHub repository, click the **cases** folder → **Add file** → **Upload files**.
2. Drag in your case file (a .txt file is best) and click **Commit changes**.
3. Click `cases.json` → ✏️ and add a line for it. For example:
   ```json
   [
     { "title": "Floppy infant, 6 weeks old", "file": "floppy-infant-6-weeks.txt" },
     { "title": "Fever and rash, 5 months old", "file": "fever-and-rash-5-months.txt" },
     { "title": "Vomiting infant, 3 weeks old", "file": "vomiting-infant-3-weeks.txt" }
   ]
   ```
   Every line ends with a comma **except the last one**. `file` must match the file name exactly. **Never put the diagnosis in the title or the file name**, because learners can see them.
4. Click **Commit changes**, wait a minute, and reload the chatbot page.

**Writing good cases:** copy `cases/case-template.txt`. You may leave the diagnosis out (the chatbot works it out) or put it on the last line for the facilitator; the peers will not reveal it early, but a learner who opens the case file will see it. Hide the real clue naturally in the history. Use **invented or fully de-identified** details only; never upload real patient records.

---

## Troubleshooting

| Problem or message | What to do |
|---|---|
| "…is not in ALLOWED_ORIGINS" | In Cloudflare set `ALLOWED_ORIGINS` to exactly `https://YOUR-USERNAME.github.io` (no repository name), then **Deploy**. |
| "Wrong access code." | Retype it in ⚙ Connection (capital letters matter). To change it, edit the `ACCESS_CODE` secret in Cloudflare and Deploy. |
| "ANTHROPIC_API_KEY is not set" | Add the secret in Cloudflare → Settings → Variables and Secrets, then Deploy. |
| "The Worker is running old code" / "looks like old Worker code" | Repeat Part 3, step 3.3, and click **Deploy**. |
| An error mentioning "model" | Set the Text variable `MODEL_BALANCED` to `claude-sonnet-5-5` (or delete it), then Deploy. |
| "credit balance is too low" | Add credit at console.anthropic.com → Billing. |
| "Could not reach the Worker" | Check the Worker address in ⚙ Connection. Open it in a tab: it should show `"ok":true`. |
| "Could not start: …" | Click Start again. If it keeps failing, choose **Thorough** quality or use a shorter case file. |
| A **Try again** button appears in the chat | The answer did not arrive. Click **Try again**. |
| The microphone does not work | Use Chrome or Edge. Click the 🔒 icon left of the web address → **Microphone: Allow** → reload. You can always type. |
| The microphone hears the peers | Use a headset, or untick **Hands-free** and click 🎤 yourself when it is your turn. |
| My answer is sent too early | In Step 4, choose a longer pause (3.5 or 5 seconds). |
| No sound from the peers | Tick **Peers speak aloud**, turn up the volume, click **🔊 Test the peers' voices**. |
| All peers sound the same, or a peer's voice does not match their gender | Choose a voice in that peer's card (voices of the right gender are listed first). A warning "No male (or female) voice is installed" means your computer has none, so the page lowers or raises the pitch instead. To add voices: **Windows** Settings → Time & language → Speech → Add voices (for example female "Microsoft Zira" or "Sonia", male "Microsoft David" or "Ryan"). **Mac** System Settings → Accessibility → Spoken Content → System voice → Manage Voices (for example female "Samantha", male "Daniel"). In **Chrome**, "Google UK English Female" and "Google UK English Male" are usually available. Reload the page afterwards. |
| A male peer (for example John) does not sound like a man | Look at his card: under **Voice** it says which voice is used. **Automatic** picks the most natural male voice available: "Google UK English Male" in Chrome, "Microsoft Ryan" or "David" in Edge or Windows, "Daniel" on a Mac. Robotic novelty voices such as "Fred" are avoided. If his card shows a ⚠ warning, no male voice is installed: add one (see the row above) and reload. |
| In Claude test mode a voice sounds the wrong gender | In Step 4 choose **Browser voices**, or with **Claude's own voices** click **🔊 try** in each peer card and pick a better-matching voice. |
| A scanned PDF will not load | It is a picture of text. Save the case as .txt or .docx instead. |
| My new shared case is not in the list | Check `cases.json` for a missing comma (paste it into https://jsonlint.com), wait a minute, and reload with **Ctrl+F5** / **Cmd+Shift+R**. |
| The page did not change after an edit | GitHub Pages takes 1 to 3 minutes. Then reload with **Ctrl+F5** (Windows) or **Cmd+Shift+R** (Mac). |

## Cost
GitHub Pages and Cloudflare Workers are free at this size. You pay Anthropic per use. A standard session (about 12 of your turns) with its evaluation on **Balanced** usually costs a few cents to around ten cents. **Fast** is cheaper; **Thorough** costs more. **Restart this case** and **Same case again** reuse the answer key, so they cost a little less.

## Tips for teachers
- **Group teaching:** project the page and let learners take turns being the participant. Use **Hold** for a real-room discussion, then **Continuation**.
- **Fair comparison:** use the same case, level, peer styles and length for everyone.
- **Focus a skill:** untick tones to change the feel; for example only "Empathetic and ethical" plus "Collaborative" for a communication-focused session.
- Copy or download each learner's report to track progress over time.
- The answer key is kept in the page's memory during a session. A technically skilled learner could find it with the browser's developer tools, so for formal assessment, run the session on your own computer.

---

## Appendix: build a new Worker from scratch
Use this only if you have no Worker, or if Part 8 told you to give Peer Case Rounds its own Worker.
1. Go to **https://dash.cloudflare.com** (no account? click **Sign up**, it is free) → left menu **Compute (Workers)** → **Workers & Pages** → **Create** → **Create Worker** (or **Start with Hello World!**).
2. Name it `infant-case-rounds` (or `peer-rounds` for a second Worker) → **Deploy**.
3. Click **Edit code**, replace everything with `worker.js` (as in step 3.3) → **Deploy**.
4. Go back → **Settings** → **Variables and Secrets** → add the three variables from step 3.4 → **Deploy**.
5. Test it as in step 3.5, then put the new address into `index.html` (step 4.3) and into ⚙ Connection.
