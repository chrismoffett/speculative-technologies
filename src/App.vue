<template>
  <div id="app">

    <!-- CARD EDITOR -->
    <div v-if="editorOpen" class="editor-overlay">
      <div class="editor-panel">
        <div class="editor-header">
          <div class="editor-title">Card Editor</div>
          <div class="btn-dark btn-dark-sm" @click="editorOpen = false">Close ✕</div>
        </div>

        <div class="editor-tabs">
          <div
            v-for="tab in editorTabs" :key="tab.key"
            class="editor-tab" :class="{ active: editorTab === tab.key }"
            @click="editorTab = tab.key">
            {{ tab.label }}
          </div>
        </div>

        <!-- Culture -->
        <div v-if="editorTab === 'culture'" class="editor-list">
          <div class="editor-item" v-for="(item, i) in editableCulture" :key="i">
            <input type="text" v-model="editableCulture[i]" />
            <div class="btn-del" @click="editableCulture.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableCulture.push('')">+ Add</div>
        </div>

        <!-- Arc Eastern -->
        <div v-if="editorTab === 'arc-eastern'" class="editor-list">
          <div class="editor-section-note">Shared futures appended automatically below.</div>
          <div class="editor-item" v-for="(item, i) in editableArcs.Eastern" :key="i">
            <input type="text" v-model="editableArcs.Eastern[i]" />
            <div class="btn-del" @click="editableArcs.Eastern.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableArcs.Eastern.push('')">+ Add</div>
        </div>

        <!-- Arc Western -->
        <div v-if="editorTab === 'arc-western'" class="editor-list">
          <div class="editor-section-note">Shared futures appended automatically below.</div>
          <div class="editor-item" v-for="(item, i) in editableArcs.Western" :key="i">
            <input type="text" v-model="editableArcs.Western[i]" />
            <div class="btn-del" @click="editableArcs.Western.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableArcs.Western.push('')">+ Add</div>
        </div>

        <!-- Arc Planetary -->
        <div v-if="editorTab === 'arc-planetary'" class="editor-list">
          <div class="editor-section-note">Shared futures appended automatically below.</div>
          <div class="editor-item" v-for="(item, i) in editableArcs.Planetary" :key="i">
            <input type="text" v-model="editableArcs.Planetary[i]" />
            <div class="btn-del" @click="editableArcs.Planetary.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableArcs.Planetary.push('')">+ Add</div>
        </div>

        <!-- Shared Futures -->
        <div v-if="editorTab === 'arc-futures'" class="editor-list">
          <div class="editor-section-note">These appear at the end of all three Arc lists.</div>
          <div class="editor-item" v-for="(item, i) in editableFutures" :key="i">
            <input type="text" v-model="editableFutures[i]" />
            <div class="btn-del" @click="editableFutures.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableFutures.push('')">+ Add</div>
        </div>

        <!-- Terrain -->
        <div v-if="editorTab === 'terrain'" class="editor-list">
          <div class="editor-item" v-for="(item, i) in editableTerrain" :key="i">
            <input type="text" v-model="editableTerrain[i]" />
            <div class="btn-del" @click="editableTerrain.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableTerrain.push('')">+ Add</div>
        </div>

        <!-- Object -->
        <div v-if="editorTab === 'object'" class="editor-list">
          <div class="editor-section-note">Include the article ("a" or "an") with each entry.</div>
          <div class="editor-item" v-for="(item, i) in editableObjects" :key="i">
            <input type="text" v-model="editableObjects[i]" />
            <div class="btn-del" @click="editableObjects.splice(i,1)">✕</div>
          </div>
          <div class="editor-add" @click="editableObjects.push('')">+ Add</div>
        </div>

        <div class="editor-footer">
          <div class="btn-dark btn-dark-sm" @click="saveEdits">Save Changes</div>
          <div class="editor-note">Changes apply immediately and persist for this session.</div>
        </div>
      </div>
    </div>

    <!-- MAIN APP -->
    <header>
      <div class="rule-heavy"></div>
      <div class="header-inner">
        <h1>Speculative<br>Technologies</h1>
        <div class="tagline">For entering worlds that are not yet, or never were.</div>
      </div>
      <div class="rule-light"></div>
    </header>

    <main>

      <div class="legend">
        <div class="legend-item" v-for="(type, key) in cardMeta" :key="key">
          <span class="legend-glyph">{{ type.glyph }}</span>
          <span class="legend-label">{{ type.label }}</span>
          <span class="legend-desc">{{ type.description }}</span>
        </div>
      </div>

      <div class="cards-row">
        <div class="card" v-for="c in displayCards" :key="c.key">
          <div class="card-type-label"><span class="glyph">{{ c.glyph }}</span>{{ c.label }}</div>
          <div class="cv" :class="{ live: hasDrawn }">{{ hasDrawn ? c.value : '—' }}</div>
        </div>
      </div>

      <div class="action-row">
        <div v-if="!committed" class="btn-dark btn-dark-lg" @click="drawCards">
          {{ hasDrawn ? 'Redeal' : 'Deal' }}
        </div>
        <div v-if="hasDrawn && !committed" class="btn-dark btn-dark-sm" @click="commit">Accept Deal →</div>
        <div v-if="committed" class="btn-outline" @click="reset">← Redeal</div>
      </div>

      <div class="sentence-banner">
        <div class="sentence-text" :class="{ live: hasDrawn }">
          <template v-if="hasDrawn">
            In a
            <span class="dt culture">{{ drawn.culture }}</span>
            <span class="dt arc">{{ drawn.arc }}</span>,
            there is
            <span class="dt object">{{ drawn.object }}</span>
            related to
            <span class="dt terrain">{{ drawn.terrain }}</span>.
          </template>
          <template v-else>In a — —, there is — related to —.</template>
        </div>
      </div>

      <!-- BRAINSTORM -->
      <section class="brainstorm" v-if="committed" ref="brainstormEl">
        <div class="bs-divider"></div>
        <div class="bs-label-row">
          <span class="bs-label">Brainstorm Sheet</span>
        </div>

        <div class="bs-sec">
          <div class="bs-n">1</div>
          <h2>First Readings</h2>
          <textarea v-model="notes.readings" rows="5" placeholder="Quick associations — real objects, historical examples, fictional technologies, gut responses…"></textarea>
        </div>

        <div class="bs-sec">
          <div class="bs-n">2</div>
          <h2>Variations <span class="h2-paren">(e.g., monumental or miniature, ritual or ruin, virtual or material, urgent or absurd, something that refuses to be useful, etc.)</span></h2>
          <div class="variations">
            <div class="var-item" v-for="(v, i) in variations" :key="i">
              <div class="var-label">{{ String.fromCharCode(65 + i) }}</div>
              <textarea v-model="variations[i]" rows="3"></textarea>
            </div>
            <div class="btn-add-var" @click="addVariation" v-if="variations.length < 8">+ Add variation</div>
          </div>
        </div>

        <div class="bs-sec">
          <div class="bs-n">3</div>
          <h2>Cultural & Historical Anchors</h2>
          <textarea v-model="notes.anchors" rows="5" placeholder="Real technologies, practices, or concepts from the Arc and Culture — resonances, inversions, critiques…"></textarea>
        </div>

        <div class="bs-sec">
          <div class="bs-n">4</div>
          <h2>What Does It Do? What Does It Say?</h2>
          <div class="two-col">
            <div>
              <label>It does:</label>
              <textarea v-model="notes.does" rows="5" placeholder="Function, operation, behavior…"></textarea>
            </div>
            <div>
              <label>It says:</label>
              <textarea v-model="notes.says" rows="5" placeholder="Argument, affect, provocation…"></textarea>
            </div>
          </div>
        </div>

        <div class="bs-sec">
          <div class="bs-n">5</div>
          <h2>Select & Commit</h2>
          <label>Working title</label>
          <input type="text" v-model="notes.title" placeholder="Working title of your speculative technology…" />
          <label>Why this direction</label>
          <textarea v-model="notes.reason" rows="5" placeholder="I'm choosing this because…"></textarea>
        </div>

        <div class="download-zone">
          <div class="btn-dark btn-dark-sm" @click="downloadNotes">Download Notes</div>
          <div class="btn-outline" @click="printNotes">Print / Save as PDF</div>
        </div>
      </section>

    </main>

    <footer>
      <div class="rule-light"></div>
      <div class="footer-inner">
        <div class="footer-row">
          <span>Speculative Technologies · MSTU 5199 · Spring 2026</span>
          <span>Based on <a class="footer-link" href="https://situationlab.org/project/the-thing-from-the-future/" target="_blank"><em>The Thing From The Future</em></a> (Situation Lab)</span>
        </div>
        <div class="footer-row">
          <span>Prof. Chris Moffett, Teachers College, Columbia University</span>
          <span class="footer-editor-link" @click="openEditor">Edit Card Deck</span>
        </div>
      </div>
    </footer>

  </div>
</template>

<script setup>
import { ref, computed, nextTick, reactive } from 'vue'
import { cards, arcsByCulture } from './cards.js'

// ── Editor state ──
const editorOpen = ref(false)
const editorTab = ref('culture')

const editorTabs = [
  { key: 'culture',      label: 'Culture' },
  { key: 'arc-eastern',  label: 'Arc: Eastern' },
  { key: 'arc-western',  label: 'Arc: Western' },
  { key: 'arc-planetary',label: 'Arc: Planetary' },
  { key: 'arc-futures',  label: 'Shared Futures' },
  { key: 'terrain',      label: 'Terrain' },
  { key: 'object',       label: 'Object' },
]

// Deep copies for editing
const editableCulture  = ref([...cards.culture.items])
const editableArcs     = reactive({
  Eastern:  [...arcsByCulture.Eastern],
  Western:  [...arcsByCulture.Western],
  Planetary:[...arcsByCulture.Planetary],
})
// Extract shared futures from Eastern (they're appended identically to all three)
// We store them separately for editing
import { sharedFuturesExport } from './cards.js'
const editableFutures  = ref([...sharedFuturesExport])
const editableTerrain  = ref([...cards.terrain.items])
const editableObjects  = ref([...cards.object.items])

// Live card data (starts from imported, overwritten on save)
const liveCards = reactive({
  culture:  [...cards.culture.items],
  arcs:     {
    Eastern:  [...arcsByCulture.Eastern],
    Western:  [...arcsByCulture.Western],
    Planetary:[...arcsByCulture.Planetary],
  },
  futures:  [...sharedFuturesExport],
  terrain:  [...cards.terrain.items],
  objects:  [...cards.object.items],
})

function openEditor() {
  // Sync editable copies from live data
  editableCulture.value  = [...liveCards.culture]
  editableArcs.Eastern   = [...liveCards.arcs.Eastern]
  editableArcs.Western   = [...liveCards.arcs.Western]
  editableArcs.Planetary = [...liveCards.arcs.Planetary]
  editableFutures.value  = [...liveCards.futures]
  editableTerrain.value  = [...liveCards.terrain]
  editableObjects.value  = [...liveCards.objects]
  editorTab.value = 'culture'
  editorOpen.value = true
}

function saveEdits() {
  liveCards.culture         = editableCulture.value.filter(s => s.trim())
  liveCards.arcs.Eastern    = editableArcs.Eastern.filter(s => s.trim())
  liveCards.arcs.Western    = editableArcs.Western.filter(s => s.trim())
  liveCards.arcs.Planetary  = editableArcs.Planetary.filter(s => s.trim())
  liveCards.futures         = editableFutures.value.filter(s => s.trim())
  liveCards.terrain         = editableTerrain.value.filter(s => s.trim())
  liveCards.objects         = editableObjects.value.filter(s => s.trim())
  editorOpen.value = false
}

// ── Game state ──
const hasDrawn  = ref(false)
const committed = ref(false)
const brainstormEl = ref(null)
const drawn = ref({ culture: '', arc: '', terrain: '', object: '' })

const cardMeta = {
  culture: { glyph: cards.culture.glyph, label: cards.culture.label, description: cards.culture.description },
  arc:     { glyph: cards.arc.glyph,     label: cards.arc.label,     description: cards.arc.description },
  object:  { glyph: cards.object.glyph,  label: cards.object.label,  description: cards.object.description },
  terrain: { glyph: cards.terrain.glyph, label: cards.terrain.label, description: cards.terrain.description },
}

function pick(arr) { return arr[Math.floor(Math.random() * arr.length)] }

function drawCards() {
  const culture = pick(liveCards.culture)
  const arcList = [...liveCards.arcs[culture], ...liveCards.futures]
  drawn.value = {
    culture,
    arc:     pick(arcList),
    terrain: pick(liveCards.terrain),
    object:  pick(liveCards.objects),
  }
  hasDrawn.value = true
  committed.value = false
  resetNotes()
}

const displayCards = computed(() => [
  { key: 'culture', glyph: cards.culture.glyph, label: cards.culture.label, value: drawn.value.culture },
  { key: 'arc',     glyph: cards.arc.glyph,     label: cards.arc.label,     value: drawn.value.arc     },
  { key: 'object',  glyph: cards.object.glyph,  label: cards.object.label,  value: drawn.value.object  },
  { key: 'terrain', glyph: cards.terrain.glyph, label: cards.terrain.label, value: drawn.value.terrain },
])

async function commit() {
  committed.value = true
  await nextTick()
  brainstormEl.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

function reset() {
  committed.value = false
  hasDrawn.value = false
  resetNotes()
}

// ── Brainstorm ──
const notes = ref({ readings: '', anchors: '', does: '', says: '', title: '', reason: '' })
const variations = ref(['', '', '', ''])

function addVariation() {
  if (variations.value.length < 8) variations.value.push('')
}

function resetNotes() {
  notes.value = { readings: '', anchors: '', does: '', says: '', title: '', reason: '' }
  variations.value = ['', '', '', '']
}

function buildText() {
  const d = drawn.value
  const varText = variations.value.map((v, i) => `  ${String.fromCharCode(65+i)}. ${v||'(blank)'}`).join('\n')
  return `SPECULATIVE TECHNOLOGIES\n${'─'.repeat(50)}\n\nIn a ${d.culture} ${d.arc}, there is ${d.object} related to ${d.terrain}.\n\n${'─'.repeat(50)}\n\n1. FIRST READINGS\n${notes.value.readings||'(blank)'}\n\n${'─'.repeat(50)}\n\n2. VARIATIONS\n${varText}\n\n${'─'.repeat(50)}\n\n3. CULTURAL & HISTORICAL ANCHORS\n${notes.value.anchors||'(blank)'}\n\n${'─'.repeat(50)}\n\n4. WHAT DOES IT DO?\n${notes.value.does||'(blank)'}\n\nWHAT DOES IT SAY?\n${notes.value.says||'(blank)'}\n\n${'─'.repeat(50)}\n\n5. SELECT & COMMIT\nWorking title: ${notes.value.title||'(blank)'}\n\nWhy this direction:\n${notes.value.reason||'(blank)'}\n`
}

function downloadNotes() {
  const blob = new Blob([buildText()], { type: 'text/plain' })
  const a = document.createElement('a')
  a.href = URL.createObjectURL(blob)
  const slug = (notes.value.title||'brainstorm').toLowerCase().replace(/\s+/g,'-').replace(/[^a-z0-9-]/g,'').slice(0,40)
  a.download = `speculative-tech-${slug}.txt`
  a.click()
  URL.revokeObjectURL(a.href)
}

function printNotes() { window.print() }
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=IM+Fell+English:ital@0;1&family=IM+Fell+English+SC&family=Space+Mono:wght@400;700&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --black: #0a0a0a;
  --dark: #1a1a1a;
  --mid: #555;
  --light: #999;
  --rule: #bbb;
  --bg: #f5f2ee;
  --paper: #faf8f4;
  --serif: 'IM Fell English', Georgia, serif;
  --serif-sc: 'IM Fell English SC', Georgia, serif;
  --mono: 'Space Mono', monospace;
  --gutter: clamp(1.5rem, 6vw, 4rem);
  --max: 760px;
}

html { font-size: 16px; }
body { background: var(--bg); color: var(--black); font-family: var(--serif); line-height: 1.6; min-height: 100vh; }

#app { display: flex; flex-direction: column; min-height: 100vh; max-width: var(--max); margin: 0 auto; padding: 0 var(--gutter); }

/* Header */
.rule-heavy { height: 3px; background: var(--black); margin-top: 1.5rem; }
.rule-light  { height: 1px; background: var(--rule); }
.header-inner { padding: 1.1rem 0 1rem; }
h1 { font-family: var(--serif); font-size: clamp(2.8rem, 10vw, 5.2rem); line-height: 0.92; font-weight: 400; font-style: italic; color: var(--black); margin-bottom: 0.4rem; }
.tagline { font-family: var(--serif); font-style: italic; font-size: 0.82rem; color: #444; }

/* Main */
main { flex: 1; padding-bottom: 4rem; }

/* Legend */
.legend { margin: 1.5rem 0 1.8rem; border-top: 1px solid var(--rule); border-bottom: 1px solid var(--rule); padding: 0.7rem 0; display: flex; flex-direction: column; gap: 0.45rem; }
.legend-item { display: grid; grid-template-columns: 1.2rem 5rem 1fr; gap: 0.5rem; align-items: baseline; }
.legend-glyph { color: var(--mid); font-size: 0.9rem; }
.legend-label { font-family: var(--mono); font-size: 0.6rem; letter-spacing: 0.08em; text-transform: uppercase; color: #333; }
.legend-desc { font-style: italic; color: #666; font-size: 0.82rem; }

/* Cards */
.cards-row { display: grid; grid-template-columns: repeat(2, 1fr); gap: 1px; background: var(--rule); border: 1px solid var(--rule); margin-bottom: 1.4rem; }
.card { background: var(--paper); padding: 1rem 0.9rem 1.25rem; min-height: 110px; display: flex; flex-direction: column; gap: 0.4rem; }
.card-type-label { font-family: var(--mono); font-size: 0.6rem; letter-spacing: 0.1em; text-transform: uppercase; color: #888; display: flex; align-items: center; gap: 0.35rem; }
.glyph { color: var(--mid); }
.cv { font-style: italic; font-size: 0.95rem; line-height: 1.4; color: #bbb; margin-top: auto; }
.cv.live { color: var(--black); }

/* Buttons */
.action-row { display: flex; gap: 0.75rem; align-items: center; margin-bottom: 1.4rem; flex-wrap: wrap; }

.btn-dark { font-family: var(--mono); font-size: 10px; letter-spacing: 0.18em; text-transform: uppercase; display: inline-block; cursor: pointer; user-select: none; }
.btn-dark-lg { background-color: #1a1a1a; color: #ffffff; padding: 0.9rem 2.6rem; box-shadow: 3px 3px 0 #555; }
.btn-dark-lg:hover { background-color: #000; }
.btn-dark-lg:active { box-shadow: none; transform: translate(2px,2px); }
.btn-dark-sm { background-color: #1a1a1a; color: #ffffff; padding: 0.7rem 1.3rem; font-size: 9px; letter-spacing: 0.12em; }
.btn-dark-sm:hover { background-color: #000; }

.btn-outline { font-family: var(--mono); font-size: 9px; letter-spacing: 0.12em; text-transform: uppercase; display: inline-block; cursor: pointer; color: var(--black); border: 1px solid var(--black); padding: 0.68rem 1.3rem; user-select: none; }
.btn-outline:hover { background-color: var(--black); color: var(--bg); }

/* Sentence */
.sentence-banner { border-top: 1px solid var(--rule); border-bottom: 1px solid var(--rule); padding: 1rem 0; margin-bottom: 0; min-height: 3.2rem; }
.sentence-text { font-family: var(--serif); font-size: clamp(0.95rem, 2.5vw, 1.15rem); line-height: 1.75; color: #bbb; font-style: italic; }
.sentence-text.live { color: var(--black); font-style: normal; }
.dt { font-style: italic; }
.dt.culture, .dt.object  { border-bottom: 2px solid var(--black); padding-bottom: 1px; }
.dt.arc,     .dt.terrain { border-bottom: 2px solid var(--mid);   padding-bottom: 1px; }

/* Brainstorm */
.brainstorm { margin-top: 0; }
.bs-divider { height: 2px; background: var(--black); margin: 2rem 0 0; }
.bs-label-row { border-bottom: 1px solid var(--rule); padding: 0.6rem 0; margin-bottom: 0; }
.bs-label { font-family: var(--mono); font-size: 9px; letter-spacing: 0.12em; text-transform: uppercase; color: var(--black); }
.bs-sec { border-bottom: 1px solid var(--rule); padding: 1.5rem 0; }
.bs-n { font-family: var(--mono); font-size: 0.6rem; color: #aaa; margin-bottom: 0.25rem; }
.bs-sec h2 { font-family: var(--serif-sc); font-size: 0.85rem; letter-spacing: 0.04em; font-weight: 400; color: #444; margin-bottom: 0.75rem; }
.h2-paren { font-family: var(--serif); font-style: italic; font-size: 0.78rem; color: #888; font-weight: 400; letter-spacing: 0; text-transform: none; }

textarea {
  width: 100%; background: var(--paper); border: 1px solid #ccc;
  padding: 0.55rem 0.7rem; font-family: var(--serif); font-size: 0.88rem;
  color: var(--black); line-height: 1.6; resize: vertical; outline: none; display: block;
}
textarea:focus { border-color: var(--mid); }
textarea::placeholder { color: #bbb; }

input[type="text"] {
  width: 100%; background: var(--paper); border: 1px solid #ccc;
  padding: 0.55rem 0.7rem; font-family: var(--serif); font-style: italic;
  font-size: 0.88rem; color: var(--black); outline: none; display: block; margin-bottom: 0.65rem;
}
input[type="text"]:focus { border-color: var(--mid); }
input[type="text"]::placeholder { color: #bbb; }

label { font-family: var(--mono); font-size: 0.6rem; letter-spacing: 0.08em; text-transform: uppercase; color: #888; display: block; margin-bottom: 0.3rem; }

.variations { display: flex; flex-direction: column; gap: 0.6rem; }
.var-item { display: flex; gap: 0.65rem; align-items: flex-start; }
.var-label { font-family: var(--mono); font-size: 0.62rem; color: #aaa; padding-top: 0.65rem; flex-shrink: 0; width: 0.9rem; }
.btn-add-var { font-family: var(--mono); font-size: 0.6rem; letter-spacing: 0.08em; text-transform: uppercase; color: #aaa; cursor: pointer; padding: 0.3rem 0; margin-left: 1.55rem; align-self: flex-start; }
.btn-add-var:hover { color: var(--mid); }

.two-col { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
@media (max-width: 520px) { .two-col { grid-template-columns: 1fr; } }

.download-zone { display: flex; gap: 0.75rem; padding: 1.5rem 0 0; flex-wrap: wrap; }

/* Footer */
footer { padding-bottom: 2rem; }
.footer-inner { padding: 0.65rem 0; font-family: var(--mono); font-size: 0.6rem; letter-spacing: 0.06em; color: #999; display: flex; flex-direction: column; gap: 0.3rem; margin-top: 1.5rem; }
.footer-row { display: flex; justify-content: space-between; flex-wrap: wrap; gap: 0.5rem; }
.footer-link { color: #999; text-decoration: underline; text-underline-offset: 2px; }
.footer-link:hover { color: var(--mid); }
.footer-editor-link { color: #999; text-decoration: underline; text-underline-offset: 2px; cursor: pointer; }
.footer-editor-link:hover { color: var(--mid); }

/* Editor overlay */
.editor-overlay {
  position: fixed; inset: 0; background: rgba(10,10,10,0.6);
  z-index: 1000; display: flex; align-items: flex-start; justify-content: center;
  padding: 2rem 1rem; overflow-y: auto;
}
.editor-panel {
  background: var(--bg); width: 100%; max-width: 680px;
  border: 2px solid var(--black); display: flex; flex-direction: column;
}
.editor-header {
  display: flex; justify-content: space-between; align-items: center;
  padding: 1rem 1.25rem; border-bottom: 1px solid var(--rule);
}
.editor-title { font-family: var(--mono); font-size: 0.7rem; letter-spacing: 0.12em; text-transform: uppercase; color: var(--black); }
.editor-tabs {
  display: flex; flex-wrap: wrap; gap: 0; border-bottom: 1px solid var(--rule);
  padding: 0.5rem 1.25rem 0;
}
.editor-tab {
  font-family: var(--mono); font-size: 0.62rem; letter-spacing: 0.08em; text-transform: uppercase;
  padding: 0.35rem 0.75rem 0.4rem; cursor: pointer; color: #999; border-bottom: 2px solid transparent;
  margin-bottom: -1px;
}
.editor-tab:hover { color: var(--black); }
.editor-tab.active { color: var(--black); border-bottom-color: var(--black); }

.editor-list { padding: 1rem 1.25rem; display: flex; flex-direction: column; gap: 0.4rem; max-height: 55vh; overflow-y: auto; }
.editor-section-note { font-family: var(--serif); font-style: italic; font-size: 0.8rem; color: #888; margin-bottom: 0.5rem; }
.editor-item { display: flex; gap: 0.5rem; align-items: center; }
.editor-item input[type="text"] { margin-bottom: 0; flex: 1; font-size: 0.82rem; padding: 0.4rem 0.6rem; }
.btn-del { font-family: var(--mono); font-size: 0.65rem; color: #bbb; cursor: pointer; flex-shrink: 0; padding: 0.2rem 0.3rem; user-select: none; }
.btn-del:hover { color: var(--black); }
.editor-add { font-family: var(--mono); font-size: 0.62rem; letter-spacing: 0.08em; text-transform: uppercase; color: #aaa; cursor: pointer; padding: 0.3rem 0; margin-top: 0.2rem; }
.editor-add:hover { color: var(--mid); }
.editor-footer {
  display: flex; align-items: center; gap: 1rem; padding: 1rem 1.25rem;
  border-top: 1px solid var(--rule); flex-wrap: wrap;
}
.editor-note { font-family: var(--serif); font-style: italic; font-size: 0.78rem; color: #888; }

/* Print */
@media print {
  header .tagline, .action-row, .bs-label-row, .download-zone, footer, .editor-overlay { display: none !important; }
  .bs-sec { break-inside: avoid; }
  body, #app { background: white; max-width: 100%; padding: 0 1rem; }
}
</style>
