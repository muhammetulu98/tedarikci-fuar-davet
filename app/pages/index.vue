<script setup lang="ts">
type Status = 'aday' | 'gm' | 'ust' | 'ok' | 'red'
interface Sevk { birim: 'Baremsiz' | 'Koli' | 'Palet'; tutar: number; alt: number; teslim: 'timon' | 'yerinden'; nakliye: 'haric' | 'dahil' }
interface Barem { alt: number; ust: number; prim: number }
interface Prim { baremli: boolean; sabit: number; baremler: Barem[] }
interface Altin { ciro: number; adet: number; tur: string }
interface Kisi { ad: string; tc: string; dogum: string }
interface Oda { tip: string; giris: string; cikis: string; kisiler: Kisi[] }
interface Detay {
  vadeAciklama: string
  iskGrubu: string
  eurKuru: number
  siparisSorumlu: { ad: string; mail: string; tel: string }
  sevkiyat: Sevk[]
  vadeTip: 'gun' | 'tarih'; vadeGun: number; vadeTarih: string
  alisIsk: number[]; satisIsk: number; hedef: number
  personelPrim: Prim; seneSonuPrim: Prim; bozukIade: number; ciroPrim: Prim; katilimPrim: Prim
  altinlar: Altin[]
  odalar: Oda[]
  duvarMt: number; masaSayisi: number; rafMt: number
  krediKarti: number; faturaKatilim: number; not: string
}
const bos = (): Detay => ({
  vadeAciklama: '',
  iskGrubu: '',
  eurKuru: 55,
  siparisSorumlu: { ad: '', mail: '', tel: '' },
  sevkiyat: [{ birim: 'Palet', tutar: 0, alt: 0, teslim: 'timon', nakliye: 'haric' }],
  vadeTip: 'gun', vadeGun: 90, vadeTarih: '', alisIsk: [0, 0, 0, 0], satisIsk: 0, hedef: 0,
  personelPrim: { baremli: false, sabit: 0, baremler: [{ alt: 0, ust: 0, prim: 0 }] },
  seneSonuPrim: { baremli: false, sabit: 0, baremler: [{ alt: 0, ust: 0, prim: 0 }] },
  bozukIade: 0,
  ciroPrim: { baremli: false, sabit: 0, baremler: [{ alt: 0, ust: 0, prim: 0 }] },
  katilimPrim: { baremli: false, sabit: 0, baremler: [{ alt: 0, ust: 0, prim: 0 }] },
  altinlar: [],
  odalar: [], duvarMt: 0, masaSayisi: 0, rafMt: 0,
  krediKarti: 0, faturaKatilim: 0, not: '',
})
interface Row extends Detay {
  gerceklesen: number
  id: number; ad: string; kod: string; yetkili: string
  ciro: number; markalar: string[]; altKategoriler: string[]
  status: Status; redEvren?: string
  log: { t: string; txt: string }[]
}

const fuarlar = ['Ekim 2026 Fuar (F2026-10)', 'Mart 2027 Fuar (F2027-03)']
const tedarikciler = [
  { ad: 'SUN PLASTİK EV GEREÇLERİ SAN. VE TİC. L…', kod: '320.34.0166', yetkili: 'Eyüp Ömer Yılmaz', ciro: 8420000, markalar: ['Sun Plastik', 'Sunplast'], altKategoriler: ['Saklama Kabı', 'Çöp Kovası', 'Plastik Mutfak'] },
  { ad: 'EMRE GIDA PAZ. SAN. VE DIŞ TİC. LTD. ŞTİ', kod: '320.34.0096', yetkili: 'Eyüp Ömer Yılmaz', ciro: 5310000, markalar: ['Emre Gıda'], altKategoriler: ['Kuru Gıda', 'Baharat'] },
  { ad: 'PROVEL TÜKETİM ÜRÜNLERİ SAN. TİC. A.Ş.', kod: '320.34.00816', yetkili: 'Eyüp Ömer Yılmaz', ciro: 2210000, markalar: ['Provel', 'Provel Home'], altKategoriler: ['Temizlik', 'Kişisel Bakım'] },
  { ad: 'KOÇYİĞİTLER GRUP İNŞ. VE SU ARM. SAN. V…', kod: '320.34.01195', yetkili: 'Eyüp Ömer Yılmaz', ciro: 3980000, markalar: ['Koçyiğit'], altKategoriler: ['Su Armatürü', 'Banyo Aksesuarı'] },
  { ad: 'SARINA CAM PAZARLAMA TİC. VE SAN. LTD.…', kod: '320.00.001479', yetkili: 'Eyüp Ömer Yılmaz', ciro: 4760000, markalar: ['Sarina'], altKategoriler: ['Cam Bardak', 'Cam Tabak', 'Kavanoz'] },
  { ad: 'MENBA DIŞ TİC. VE SAN. LTD. ŞTİ.', kod: '320.00.001398', yetkili: 'Eyüp Ömer Yılmaz', ciro: 6120000, markalar: ['Menba'], altKategoriler: ['Mutfak Gereçleri'] },
  { ad: 'QLUX IDEAS MUTFAK EŞYA. SAN. VE TİC. A…', kod: '320.00.001336', yetkili: 'Eyüp Ömer Yılmaz', ciro: 940000, markalar: ['Qlux', 'Ideas'], altKategoriler: ['Mutfak Eşyası', 'Termos'] },
  { ad: 'SİGMA CAM EŞYA PAZ. TİC. LTD. ŞTİ.', kod: '320.34.08872', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Sigma'], altKategoriler: ['Cam Eşya', 'Sürahi'] },
  { ad: 'AKAY PLASTİK VE SAN. TİC. LTD. ŞTİ.', kod: '320.34.08960', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Akay'], altKategoriler: ['Plastik Kova', 'Sepet'] },
  { ad: 'ORCAMP E-TİCARET LTD. ŞTİ.', kod: '320.00.001352', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Orcamp'], altKategoriler: ['Kamp Malzemesi'] },
  { ad: 'MENZİR MADENİ EŞYA PLASTİK TEKS. GIDA …', kod: '320.00.001669', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Menzir'], altKategoriler: ['Çatal Kaşık', 'Tencere', 'Plastik Mutfak'] },
  { ad: 'BURSEV PLASTİK VE DIŞ TİC. A.Ş', kod: '320.00.000101', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Bursev'], altKategoriler: ['Plastik Ev Gereçleri'] },
  { ad: 'ÖZÇELİK AYNA CAM SAN. TİC.LTD.ŞTİ.', kod: '320.00.001666', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Özçelik'], altKategoriler: ['Ayna', 'Cam Bardak'] },
  { ad: 'DURUL TEKSTİL KONFEKSİYON İNŞ.GIDA EL…', kod: '320.00.001610', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Durul'], altKategoriler: ['Ev Tekstili', 'Mutfak Tekstili'] },
  { ad: 'ÜÇEF ENDÜSTRİYEL MUTFAK SAN. İTH. VE İ…', kod: '320.00.001371', yetkili: 'Eyüp Ömer Yılmaz', ciro: 0, markalar: ['Üçef'], altKategoriler: ['Endüstriyel Mutfak', 'Tencere'] },
]

const now = () => new Date().toLocaleString('tr-TR', { dateStyle: 'short', timeStyle: 'short' })
const mk = (i: number, status: Status, o: Partial<Row>): Row => ({
  id: i, ...tedarikciler[i - 1]!, ...bos(), gerceklesen: 0, status, log: [{ t: '26.09.2026 10:12', txt: 'Aday olarak eklendi' }], ...o,
})
const rows = ref<Row[]>([
  mk(1, 'ust', { gerceklesen: 2150000, krediKarti: 500000, faturaKatilim: 120000, alisIsk: [12, 3, 0, 0], satisIsk: 6, vadeGun: 120, odalar: [{ tip: 'DBL', giris: '2026-10-14', cikis: '2026-10-18', kisiler: [{ ad: 'Ahmet Demir', tc: '', dogum: '' }, { ad: 'Mehmet Kaya', tc: '', dogum: '' }] }], duvarMt: 12, masaSayisi: 6, rafMt: 20, personelPrim: { baremli: true, sabit: 0, baremler: [{ alt: 0, ust: 500000, prim: 1 }, { alt: 500000, ust: 1000000, prim: 2 }] }, ciroPrim: { baremli: false, sabit: 1.5, baremler: [{ alt: 0, ust: 0, prim: 0 }] }, hedef: 3000000 }),
  mk(2, 'gm', { krediKarti: 300000, faturaKatilim: 85000, alisIsk: [10, 3, 0, 0], satisIsk: 5, vadeGun: 90, odalar: [{ tip: 'DBL', giris: '2026-10-14', cikis: '2026-10-18', kisiler: [{ ad: 'Ahmet Demir', tc: '', dogum: '' }, { ad: 'Mehmet Kaya', tc: '', dogum: '' }] }], duvarMt: 12, masaSayisi: 6, rafMt: 20, personelPrim: { baremli: true, sabit: 0, baremler: [{ alt: 0, ust: 500000, prim: 1 }, { alt: 500000, ust: 1000000, prim: 2 }] }, ciroPrim: { baremli: false, sabit: 1.5, baremler: [{ alt: 0, ust: 0, prim: 0 }] }, hedef: 2000000 }),
  mk(3, 'aday', {}),
  mk(4, 'ok', { gerceklesen: 1340000, krediKarti: 200000, faturaKatilim: 60000, alisIsk: [9, 3, 0, 0], satisIsk: 4.5, vadeGun: 90, odalar: [{ tip: 'DBL', giris: '2026-10-14', cikis: '2026-10-18', kisiler: [{ ad: 'Ahmet Demir', tc: '', dogum: '' }, { ad: 'Mehmet Kaya', tc: '', dogum: '' }] }], duvarMt: 12, masaSayisi: 6, rafMt: 20, personelPrim: { baremli: true, sabit: 0, baremler: [{ alt: 0, ust: 500000, prim: 1 }, { alt: 500000, ust: 1000000, prim: 2 }] }, ciroPrim: { baremli: false, sabit: 1.5, baremler: [{ alt: 0, ust: 0, prim: 0 }] }, hedef: 1200000 }),
  mk(5, 'red', { krediKarti: 250000, faturaKatilim: 70000, alisIsk: [8, 3, 0, 0], satisIsk: 3, vadeGun: 60, odalar: [{ tip: 'DBL', giris: '2026-10-14', cikis: '2026-10-18', kisiler: [{ ad: 'Ahmet Demir', tc: '', dogum: '' }, { ad: 'Mehmet Kaya', tc: '', dogum: '' }] }], duvarMt: 12, masaSayisi: 6, rafMt: 20, personelPrim: { baremli: true, sabit: 0, baremler: [{ alt: 0, ust: 500000, prim: 1 }, { alt: 500000, ust: 1000000, prim: 2 }] }, ciroPrim: { baremli: false, sabit: 1.5, baremler: [{ alt: 0, ust: 0, prim: 0 }] }, hedef: 1500000, redEvren: "Yönetim" }),
])

const stages = [
  { key: 'aday', no: '01', who: 'Sistem / Admin', name: 'Aday Listesi', color: 'var(--faint)', match: (r: Row) => true },
  { key: 'yon', no: '02', who: 'Genel Müdür / Üst Yönetim', name: 'Yönetim Onayı', color: 'var(--orange)', match: (r: Row) => r.status === 'gm' || r.status === 'ust' },
]
const stageFilter = ref<string | null>(null)
const fuar = ref(fuarlar[0])
const seciliTed = ref('')
const durum = ref('')
const ara = ref('')
const selected = ref<number[]>([])
const toastMsg = ref('')

const statusMeta: Record<Status, { label: string; cls: string; row: string }> = {
  aday: { label: 'Aday', cls: 'b-aday', row: '' },
  gm: { label: 'Yönetim Onayında', cls: 'b-gm', row: 's-gm' },
  ust: { label: 'Üst Yönetim Onayında', cls: 'b-ust', row: 's-ust' },
  ok: { label: 'Onaylandı', cls: 'b-ok', row: 's-ok' },
  red: { label: 'Reddedildi', cls: 'b-red', row: 's-red' },
}

const visible = computed(() =>
  rows.value.filter(r => {
    if (stageFilter.value) {
      const s = stages.find(x => x.key === stageFilter.value)!
      if (!s.match(r)) return false
    }
    if (durum.value && r.status !== durum.value) return false
    const q = ara.value.trim().toLocaleLowerCase('tr')
    if (q && !`${r.ad} ${r.kod} ${r.yetkili} ${r.markalar.join(' ')} ${r.altKategoriler.join(' ')}`.toLocaleLowerCase('tr').includes(q)) return false
    return true
  }),
)
const count = (s: typeof stages[number]) => rows.value.filter(s.match).length
const availableTed = computed(() => tedarikciler.filter(t => !rows.value.some(r => r.kod === t.kod)))

const tedQuery = ref('')
const tedOpen = ref(false)
const tedSonuc = computed(() => {
  const q = tedQuery.value.trim().toLocaleLowerCase('tr')
  return availableTed.value.filter(t => !q || `${t.ad} ${t.kod}`.toLocaleLowerCase('tr').includes(q)).slice(0, 50)
})
function pickTed(t: typeof tedarikciler[number]) { seciliTed.value = t.kod; tedQuery.value = t.ad; tedOpen.value = false }
function closeCombo() { tedOpen.value = false }

const selRows = computed(() => rows.value.filter(r => selected.value.includes(r.id)))
const allIn = (...st: Status[]) => selRows.value.length > 0 && selRows.value.every(r => st.includes(r.status))
const allChecked = computed({
  get: () => visible.value.length > 0 && visible.value.every(r => selected.value.includes(r.id)),
  set: v => (selected.value = v ? visible.value.map(r => r.id) : []),
})

const fmt = (n: number) => '₺' + n.toLocaleString('tr-TR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
const toast = (m: string) => { toastMsg.value = m; setTimeout(() => (toastMsg.value = ''), 2200) }

function move(to: Status, txt: string, extra: Partial<Row> = {}) {
  selRows.value.forEach(r => {
    r.log.push({ t: now(), txt })
    Object.assign(r, { status: to }, extra)
  })
  toast(`${selRows.value.length} kayıt: ${txt}`)
  selected.value = []
}
function sil(list: Row[]) {
  const ids = list.filter(r => r.status === 'aday').map(r => r.id)
  if (!ids.length || !confirm(`${ids.length} aday silinsin mi?`)) return
  rows.value = rows.value.filter(r => !ids.includes(r.id))
  selected.value = selected.value.filter(i => !ids.includes(i))
  toast(`${ids.length} aday silindi`)
}
const gmeGonder = () => move('gm', 'Yönetim onayına gönderildi')
const ustGonder = () => move('ust', 'Genel Müdür lüzum gördü → Üst Yönetim onayına gönderildi')
const onay = () => move('ok', 'Yönetim onayladı — davet kesinleşti')
const red = () => move('red', 'Yönetim reddetti', { redEvren: 'Yönetim' })
const geriCek = () => move('aday', 'Adaya geri çekildi')
const sifirla = () => move('aday', 'Sıfırlandı')

// drawer
const edit = ref<Row | null>(null)
const isNew = ref(false)
const form = reactive<Detay>(bos())
const klon = <T,>(x: T): T => JSON.parse(JSON.stringify(x))
const kilitli = computed(() => !!edit.value && edit.value.status !== 'aday')

function adayUret() {
  const t = tedarikciler.find(x => x.kod === seciliTed.value)
  if (!t) return
  const r: Row = { id: Date.now(), ...t, ...bos(), gerceklesen: 0, status: 'aday', log: [{ t: now(), txt: 'Aday olarak eklendi' }] }
  rows.value.unshift(r)
  seciliTed.value = ''
  tedQuery.value = ''
  openEdit(r)
}
function openEdit(r: Row) {
  edit.value = r
  const { id, ad, kod, yetkili, ciro, markalar, altKategoriler, gerceklesen, status, log, redEvren, ...d } = r
  Object.assign(form, klon(d))
}
function kaydet(gonder = false) {
  const r = edit.value!
  Object.assign(r, klon(form))
  r.log.push({ t: now(), txt: 'Fuar bilgileri güncellendi' })
  if (gonder) { r.status = 'gm'; r.log.push({ t: now(), txt: 'Yönetim onayına gönderildi' }) }
  edit.value = null
  toast(gonder ? 'Yönetim onayına gönderildi' : 'Kaydedildi')
}
const addBarem = (k: 'personelPrim' | 'seneSonuPrim' | 'ciroPrim' | 'katilimPrim') => form[k].baremler.push({ alt: form[k].baremler.at(-1)?.ust ?? 0, ust: 0, prim: 0 })
const delBarem = (k: 'personelPrim' | 'seneSonuPrim' | 'ciroPrim' | 'katilimPrim', i: number) => { if (form[k].baremler.length > 1) form[k].baremler.splice(i, 1) }
const gece = (o: Oda) => (o.giris && o.cikis ? Math.max(0, Math.round((+new Date(o.cikis) - +new Date(o.giris)) / 864e5)) : 0)
const odaTutar = (o: Oda) => (odaFiyat[o.tip] ?? 0) * gece(o)
const konaklamaEur = computed(() => form.odalar.reduce((t, o) => t + odaTutar(o), 0))
const katilimTl = computed(() => (form.krediKarti || 0) + (form.faturaKatilim || 0))
const konaklamaTl = computed(() => konaklamaEur.value * (form.eurKuru || 0))
const duvarTl = computed(() => (form.duvarMt || 0) * gider.duvar)
const rafTl = computed(() => (form.rafMt || 0) * gider.raf)
const masaTl = computed(() => (form.masaSayisi || 0) * gider.masa)
const alanTl = computed(() => duvarTl.value + rafTl.value + masaTl.value)
const primTutar = computed(() => ((edit.value?.gerceklesen ?? 0) * (form.katilimPrim.sabit || 0)) / 100)
const masraf = computed(() => konaklamaTl.value + alanTl.value - katilimTl.value - primTutar.value)
const masrafTon = computed(() => (konaklamaTl.value + alanTl.value + katilimTl.value + primTutar.value === 0 ? 'nt' : masraf.value > 0 ? 'wr' : 'ok'))
const eur = (n: number) => '€' + n.toLocaleString('tr-TR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
const altinTurleri = ['Gram Altın', 'Çeyrek Altın', 'Yarım Altın', 'Tam Altın']
const addAltin = () => form.altinlar.push({ ciro: form.altinlar.at(-1)?.ciro ?? 0, adet: 1, tur: 'Gram Altın' })
const addSevk = () => form.sevkiyat.push({ birim: 'Palet', tutar: 0, alt: 0, teslim: 'timon', nakliye: 'haric' })
const delSevk = (i: number) => { if (form.sevkiyat.length > 1) form.sevkiyat.splice(i, 1) }
const yeniKisi = (): Kisi => ({ ad: '', tc: '', dogum: '' })
const addOda = () => form.odalar.push({ tip: '', giris: '', cikis: '', kisiler: [yeniKisi()] })
const delOda = (i: number) => form.odalar.splice(i, 1)
const gider = reactive({
  oda: { SNG: 243, DBL: 324, 'DBL+1': 405, TRPL: 454, 'TRPL+1': 535, FAM: 648 } as Record<string, number>,
  duvar: 0, raf: 0, masa: 0,
})
const odaFiyat = gider.oda
const giderOpen = ref(false)
const odaTipleri = Object.keys(odaFiyat)
const iskTxt = (a: number[]) => a.filter(Boolean).map(x => '%' + x).join(' + ') || '—'
const tr = (d: string) => new Date(d).toLocaleDateString('tr-TR')
const vadeTxt = (r: Detay) => r.vadeTip === 'gun' ? `${r.vadeGun} gün` : (r.vadeTarih ? tr(r.vadeTarih) : '—')
</script>

<template>
  <div class="page">
    <div class="head">
      <div>
        <h1>Fuar Davet — Tedarikçi</h1>
        <p>Fuara davet edilecek tedarikçileri seçin, fuar koşullarını girin ve yönetim onayına gönderin (gerekirse Genel Müdür üst yönetime iletir).</p>
      </div>
      <div class="actions">
        <button class="btn" @click="giderOpen = true">Fuar Masraf Tablosu</button>
        <button class="btn" @click="toast('Excel’e aktarıldı (prototip)')">Excel’e Aktar</button>
        <button class="btn" @click="toast('Excel’den karar aktarıldı (prototip)')">Excel’den Karar Aktar</button>
      </div>
    </div>

    <div class="stages">
      <button v-for="s in stages" :key="s.key" class="stage" :class="{ on: stageFilter === s.key }"
        @click="stageFilter = stageFilter === s.key ? null : s.key">
        <div class="no"><span class="dot" :style="{ background: s.color }" /> {{ s.no }}</div>
        <div class="who">{{ s.who }}</div>
        <div class="row"><span class="name">{{ s.name }}</span><span class="cnt">{{ count(s) }}</span></div>
      </button>
    </div>
    <div class="hint">Bir aşama kartına tıklayarak listeyi o aşamadaki tedarikçilere göre filtreleyin.</div>

    <div class="panel">
      <div class="filters">
        <div class="field"><label>Proje / Fuar</label>
          <select v-model="fuar" class="inp w-lg"><option v-for="f in fuarlar" :key="f">{{ f }}</option></select></div>
        <div class="field"><label>Tedarikçi <i>*</i></label>
          <div class="combo">
            <input v-model="tedQuery" class="inp w-lg" placeholder="Tedarikçi adı veya cari kodu ara…"
              @focus="tedOpen = true" @input="tedOpen = true; seciliTed = ''" @blur="closeCombo">
            <ul v-if="tedOpen" class="combo-list">
              <li v-for="t in tedSonuc" :key="t.kod" @mousedown.prevent="pickTed(t)">
                <span>{{ t.ad }}</span><small>{{ t.kod }}</small>
              </li>
              <li v-if="!tedSonuc.length" class="none">Sonuç bulunamadı</li>
            </ul>
          </div></div>
        <button class="btn btn-dark" :disabled="!seciliTed" @click="adayUret">Aday Ekle</button>
        <div class="grow" />
        <div class="field"><label>Durum</label>
          <select v-model="durum" class="inp">
            <option value="">Tüm durumlar</option>
            <option v-for="(m, k) in statusMeta" :key="k" :value="k">{{ m.label }}</option>
          </select></div>
        <div class="field"><label>Ara</label><input v-model="ara" class="inp w-lg" placeholder="Tedarikçi, cari kodu, marka, kategori…"></div>
      </div>
    </div>

    <div class="toolbar">
      <label class="chk"><input v-model="allChecked" type="checkbox"> {{ selected.length }} seçili</label>
      <button class="btn btn-dark" :disabled="!allIn('aday')" @click="gmeGonder">Yönetim Onayına Gönder</button>
      <span class="sep" />
      <button class="btn btn-green" :disabled="!allIn('gm', 'ust')" @click="onay">Onayla</button>
      <button class="btn btn-ghost-red" :disabled="!allIn('gm', 'ust')" @click="red">Reddet</button>
      <button class="btn btn-ghost-purple" :disabled="!allIn('gm')" @click="ustGonder">Üst Yönetime Gönder</button>
      <span class="sep" />
      <button class="btn btn-ghost-red" :disabled="!allIn('aday')" @click="sil(selRows)">Sil</button>
      <button class="btn btn-ghost-orange" :disabled="!allIn('gm', 'ust')" @click="geriCek">Geri Çek</button>
      <button class="btn" :disabled="!allIn('ok', 'red')" @click="sifirla">Sıfırla</button>
    </div>

    <div class="tbl-wrap">
      <table>
        <thead>
          <tr>
            <th style="width:32px" />
            <th>Tedarikçi</th>
            <th>Markalar</th>
            <th>Alt Kategoriler</th>
            <th>Durum</th>
            <th>Sorumlu</th>
            <th class="r">Vade</th>
            <th class="r">Alış İsk.</th>
            <th class="r">Satış İsk.</th>
            <th class="r">Hedef Satış Cirosu</th>
            <th style="width:64px" />
          </tr>
        </thead>
        <tbody>
          <tr v-for="r in visible" :key="r.id" :class="statusMeta[r.status].row">
            <td><input v-model="selected" type="checkbox" :value="r.id"></td>
            <td><span class="name">{{ r.ad }}</span><span class="sub">{{ r.kod }}</span></td>
            <td><div class="chips"><span v-for="m in r.markalar" :key="m" class="chip">{{ m }}</span><span v-if="!r.markalar.length">—</span></div></td>
            <td><div class="chips"><span v-for="a in r.altKategoriler" :key="a" class="chip alt">{{ a }}</span><span v-if="!r.altKategoriler.length">—</span></div></td>
            <td><span class="badge" :class="statusMeta[r.status].cls">{{ statusMeta[r.status].label }}<template v-if="r.status === 'red' && r.redEvren"> · {{ r.redEvren }}</template></span></td>
            <td>{{ r.yetkili }}</td>
            <td class="r num">{{ vadeTxt(r) }}</td>
            <td class="r num">{{ iskTxt(r.alisIsk) }}</td>
            <td class="r num">{{ r.satisIsk ? '%' + r.satisIsk : '—' }}</td>
            <td class="r num">{{ r.hedef ? fmt(r.hedef) : '—' }}</td>
            <td style="white-space:nowrap">
              <button class="iconbtn" title="Fuar bilgileri" @click="openEdit(r)">✎</button>
              <button v-if="r.status === 'aday'" class="iconbtn del" title="Adayı sil" @click="sil([r])">🗑</button>
            </td>
          </tr>
          <tr v-if="!visible.length"><td colspan="12" class="empty">Kayıt bulunamadı. Yukarıdan tedarikçi seçip aday ekleyin.</td></tr>
        </tbody>
      </table>
    </div>

    <div v-if="edit" class="overlay" @click.self="edit = null">
      <div class="drawer">
        <header>
          <div>
            <h2>{{ edit.ad }}</h2>
            <p>{{ edit.kod }} · {{ fuar }} · <span class="badge" :class="statusMeta[edit.status].cls">{{ statusMeta[edit.status].label }}</span></p>
          </div>
          <button class="iconbtn" @click="edit = null">✕</button>
        </header>
        <div class="body">
          <div class="sec s1">1 · Alış Şartları</div>
          <div class="g2">
            <div class="field full"><label>Fuar Alış Vadesi</label>
              <div style="display:flex;gap:14px;align-items:center">
                <label class="chk"><input v-model="form.vadeTip" type="radio" value="gun" :disabled="kilitli"> Gün</label>
                <label class="chk"><input v-model="form.vadeTip" type="radio" value="tarih" :disabled="kilitli"> Nokta tarih</label>
                <div v-if="form.vadeTip === 'gun'" class="suffix"><input v-model.number="form.vadeGun" class="inp" type="number" :disabled="kilitli"><span>gün</span></div>
                <input v-else v-model="form.vadeTarih" class="inp" type="date" :disabled="kilitli">
              </div></div>
            <div class="field full"><label>Vade Açıklaması</label>
              <input v-model="form.vadeAciklama" class="inp" maxlength="120" placeholder="Kısa açıklama (ör. fuar sonrası 30 gün, aylık kapanış…)" :disabled="kilitli"></div>
          </div>

          <div class="g4">
            <div v-for="(_, i) in form.alisIsk" :key="i" class="field"><label>İskonto {{ i + 1 }}</label>
              <div class="suffix"><input v-model.number="form.alisIsk[i]" class="inp" type="number" step="0.1" :disabled="kilitli"><span>%</span></div></div>
          </div>

          <div class="field" style="margin-bottom:6px"><label>Sevkiyat Baremleri</label></div>
          <div class="sevk sevk-h"><span>Baremli</span><span>Alt sınır</span><span>Teslim yeri</span><span>Nakliye</span><span /></div>
          <div v-for="(v, i) in form.sevkiyat" :key="i" class="sevk">
            <select v-model="v.birim" class="inp" :disabled="kilitli"><option>Baremsiz</option><option>Koli</option><option>Palet</option></select>
            <div v-if="v.birim === 'Baremsiz'" class="suffix"><MoneyInput v-model="v.tutar" class="inp" placeholder="Tutar" :disabled="kilitli" /><span>₺</span></div>
            <input v-else v-model.number="v.alt" class="inp" type="number" placeholder="Alt sınır" :disabled="kilitli">
            <select v-model="v.teslim" class="inp" :disabled="kilitli"><option value="timon">Timon Depo</option><option value="yerinden">Yerinden</option></select>
            <select v-model="v.nakliye" class="inp" :disabled="kilitli"><option value="haric">Nakliye Hariç</option><option value="dahil">Nakliye Dahil</option></select>
            <button class="iconbtn" :disabled="kilitli" @click="delSevk(i)">✕</button>
          </div>
          <button v-if="!kilitli" class="btn" style="margin-bottom:14px" @click="addSevk">+ Sevkiyat Baremi Ekle</button>

          <div class="sec s2">2 · Satış İskontosu ve Hedef</div>
          <div class="g2" style="grid-template-columns:110px 110px 1fr">
            <div class="field"><label>Fuar Satış İskontosu</label>
              <div class="suffix"><input v-model.number="form.satisIsk" class="inp" type="number" step="0.1" :disabled="kilitli"><span>%</span></div></div>
            <div class="field"><label>İskonto Grubu</label>
              <select v-model="form.iskGrubu" class="inp" :disabled="kilitli">
                <option value="">Seçin</option><option>A0</option><option>A1</option><option>A2</option>
              </select></div>
            <div class="field"><label>Hedef Satış Cirosu</label>
              <div class="suffix"><MoneyInput v-model="form.hedef" class="inp" :disabled="kilitli" /><span>₺</span></div></div>
          </div>

          <div class="sec s3">3 · Primler (Baremli)</div>
          <div v-for="pr in [{ k: 'personelPrim', t: 'Personel Primi', zorunlu: false }, { k: 'seneSonuPrim', t: 'Sene Sonu Primi', zorunlu: false }, { k: 'ciroPrim', t: 'Ciro Primi', zorunlu: false }] as const" :key="pr.k" style="margin-bottom:14px">
            <div class="field" style="margin-bottom:6px"><label>{{ pr.t }}</label></div>
            <label v-if="!pr.zorunlu" class="chk" style="margin-bottom:8px"><input v-model="form[pr.k].baremli" type="checkbox" :disabled="kilitli"> Baremli</label>
            <div v-if="!pr.zorunlu && !form[pr.k].baremli" class="field" style="max-width:200px">
              <div class="suffix"><input v-model.number="form[pr.k].sabit" class="inp" type="number" step="0.1" placeholder="Prim oranı" :disabled="kilitli"><span>%</span></div></div>
            <template v-else>
            <div v-for="(b, i) in form[pr.k].baremler" :key="i" class="barem">
              <div class="suffix"><MoneyInput v-model="b.alt" class="inp" placeholder="Alt sınır" :disabled="kilitli" /><span>₺</span></div>
              <div class="suffix"><MoneyInput v-model="b.ust" class="inp" placeholder="Üst sınır" :disabled="kilitli" /><span>₺</span></div>
              <div class="suffix"><input v-model.number="b.prim" class="inp" type="number" step="0.1" placeholder="Prim" :disabled="kilitli"><span>%</span></div>
              <button class="iconbtn" :disabled="kilitli" @click="delBarem(pr.k, i)">✕</button>
            </div>
            <button v-if="!kilitli" class="btn" @click="addBarem(pr.k)">+ Barem Ekle</button>
            </template>
          </div>

          <div class="field" style="margin-bottom:14px;max-width:200px"><label>Bozuk İade Bütçesi <small style="color:var(--faint)">(baremsiz)</small></label>
            <div class="suffix"><input v-model.number="form.bozukIade" class="inp" type="number" step="0.01" :disabled="kilitli"><span>%</span></div></div>

          <div class="field" style="margin-bottom:6px"><label>Ciro Hedefine Göre Altın Ödülü</label></div>
          <div v-for="(a, i) in form.altinlar" :key="i" class="altin">
            <div class="suffix"><MoneyInput v-model="a.ciro" class="inp" placeholder="Ciro" :disabled="kilitli" /><span>₺</span></div>
            <input v-model.number="a.adet" class="inp" type="number" min="1" placeholder="Adet" :disabled="kilitli">
            <select v-model="a.tur" class="inp" :disabled="kilitli"><option v-for="t in altinTurleri" :key="t">{{ t }}</option></select>
            <button class="iconbtn" :disabled="kilitli" @click="form.altinlar.splice(i, 1)">✕</button>
          </div>
          <button v-if="!kilitli" class="btn" style="margin-bottom:14px" @click="addAltin">+ Altın Ödülü Ekle</button>

          <div class="sec s4">4 · Tedarikci Sipariş Sorumlusu</div>
          <div class="g3">
            <div class="field"><label>Ad Soyad</label><input v-model="form.siparisSorumlu.ad" class="inp" :disabled="kilitli"></div>
            <div class="field"><label>E-posta</label><input v-model="form.siparisSorumlu.mail" class="inp" type="email" :disabled="kilitli"></div>
            <div class="field"><label>Telefon</label><input v-model="form.siparisSorumlu.tel" class="inp" :disabled="kilitli"></div>
          </div>

          <div class="sec s5">5 · Konaklama</div>
          <div v-for="(o, oi) in form.odalar" :key="oi" class="oda">
            <div class="oda-h"><b>Oda {{ oi + 1 }}</b>
              <button class="iconbtn" :disabled="kilitli" @click="delOda(oi)">✕ Odayı Sil</button></div>
            <div class="g2">
              <div class="field"><label>Oda Tipi</label>
                <select v-model="o.tip" class="inp" :disabled="kilitli"><option value="">Seçin</option><option v-for="t in odaTipleri" :key="t" :value="t">{{ t }} — {{ eur(odaFiyat[t]!) }}</option></select></div>
              <div class="field"><label>Giriş – Çıkış Tarihi</label><DateRange v-model:giris="o.giris" v-model:cikis="o.cikis" :disabled="kilitli" /></div>
            </div>
            <div v-if="o.tip" class="oda-fiyat">{{ o.tip }} · {{ eur(odaFiyat[o.tip]!) }} × {{ gece(o) }} gece = <b>{{ eur(odaTutar(o)) }}</b></div>
            <div class="field" style="margin-bottom:6px"><label>Kişi Bilgileri</label></div>
            <div v-for="(k, ki) in o.kisiler" :key="ki" class="kisi">
              <input v-model="k.ad" class="inp" placeholder="Ad Soyad" :disabled="kilitli">
              <input v-model="k.tc" class="inp" placeholder="TC Kimlik No" maxlength="11" :disabled="kilitli">
              <input v-model="k.dogum" class="inp" type="date" title="Doğum tarihi" :disabled="kilitli">
              <button class="iconbtn" :disabled="kilitli || o.kisiler.length < 2" @click="o.kisiler.splice(ki, 1)">✕</button>
            </div>
            <button v-if="!kilitli" class="btn" @click="o.kisiler.push(yeniKisi())">+ Kişi Ekle</button>
          </div>
          <button v-if="!kilitli" class="btn" style="margin-bottom:14px" @click="addOda">{{ form.odalar.length ? '+ Bir Oda Daha Ekle' : '+ Oda Ekle' }}</button>

          <div class="sec s6">6 · Alan Bilgileri</div>
          <div class="g3">
            <div class="field"><label>Duvar</label><div class="suffix"><input v-model.number="form.duvarMt" class="inp" type="number" :disabled="kilitli"><span>mt</span></div></div>
            <div class="field"><label>Masa Sayısı</label><div class="suffix"><input v-model.number="form.masaSayisi" class="inp" type="number" :disabled="kilitli"><span>adet</span></div></div>
            <div class="field"><label>Raf</label><div class="suffix"><input v-model.number="form.rafMt" class="inp" type="number" :disabled="kilitli"><span>mt</span></div></div>
          </div>

          <div class="sec">Diğer</div>
          <div class="g2">
            <div class="field"><label>Fuar Kredi Kartı Katılım Bedeli</label><div class="suffix"><MoneyInput v-model="form.krediKarti" class="inp" :disabled="kilitli" /><span>₺</span></div></div>
            <div class="field"><label>Fuar Fatura Katılım Bedeli</label><div class="suffix"><MoneyInput v-model="form.faturaKatilim" class="inp" :disabled="kilitli" /><span>₺</span></div></div>
            <div class="field full"><label>Gerçekleşen Ciro <small style="color:var(--faint)">(otomatik · ERP’den gelir, manuel giriş yok)</small></label>
              <div class="suffix"><input class="inp" :value="fmt(edit.gerceklesen).replace('₺', '')" readonly disabled style="background:#f6f6f8"><span>₺</span></div></div>
            <div class="field"><label>Cirodan Fuar Katılım Primi</label><div class="suffix"><input v-model.number="form.katilimPrim.sabit" class="inp" type="number" step="0.1" :disabled="kilitli"><span>%</span></div></div>
            <div class="field full"><label>Açıklama</label>
              <textarea v-model="form.not" class="inp" rows="3" style="height:auto;padding:8px 10px;width:100%" :disabled="kilitli" /></div>
          </div>

          <div class="masraf" :class="masrafTon">
            <div>
              <span>Toplam Masraf</span><b class="num">{{ fmt(masraf) }}</b>
              <small>Konaklama {{ eur(konaklamaEur) }}<template v-if="form.eurKuru > 0"> × {{ form.eurKuru }} = {{ fmt(konaklamaTl) }}</template></small>
              <small>+ Duvar {{ form.duvarMt || 0 }} mt × {{ fmt(gider.duvar) }} = {{ fmt(duvarTl) }}</small>
              <small>+ Masa {{ form.masaSayisi || 0 }} adet × {{ fmt(gider.masa) }} = {{ fmt(masaTl) }}</small>
              <small>+ Raf {{ form.rafMt || 0 }} mt × {{ fmt(gider.raf) }} = {{ fmt(rafTl) }}</small>
              <small>− Kredi kartı katılım {{ fmt(form.krediKarti) }} − Fatura katılım {{ fmt(form.faturaKatilim) }} − Katılım primi {{ fmt(primTutar) }} (gerçekleşen ciro × %{{ form.katilimPrim.sabit || 0 }})</small>
              <small v-if="konaklamaEur > 0 && !(form.eurKuru > 0)">Konaklama toplama girmesi için EUR kuru girin.</small>
            </div>
            <div class="r"><span>EUR Kuru</span>
              <div class="suffix" style="width:110px;margin-left:auto"><input v-model.number="form.eurKuru" class="inp" type="number" step="0.01" disabled><span>₺</span></div></div>
          </div>

          <div class="sec">Onay Geçmişi</div>
          <div class="timeline">
            <div v-for="(l, i) in [...edit.log].reverse()" :key="i"><b>{{ l.t }}</b> — {{ l.txt }}</div>
          </div>
        </div>
        <footer>
          <button class="btn" @click="edit = null">Kapat</button>
          <template v-if="!kilitli">
            <button class="btn" @click="kaydet()">Kaydet</button>
            <button class="btn btn-dark" @click="kaydet(true)">Kaydet ve Yönetim Onayına Gönder</button>
          </template>
        </footer>
      </div>
    </div>

    <div v-if="giderOpen" class="overlay" @click.self="giderOpen = false">
      <div class="drawer" style="width:460px">
        <header>
          <div><h2>Fuar Masraf Gider Tablosu</h2><p>Buradaki fiyatlar tüm adayların masraf hesabında kullanılır.</p></div>
          <button class="iconbtn" @click="giderOpen = false">✕</button>
        </header>
        <div class="body">
          <div class="sec s1">1 · Oda Fiyatları <small style="text-transform:none;letter-spacing:0">(gecelik, €)</small></div>
          <div class="g2">
            <div v-for="(_, t) in gider.oda" :key="t" class="field"><label>{{ t }}</label>
              <div class="suffix"><MoneyInput v-model="gider.oda[t]" class="inp" /><span>€</span></div></div>
          </div>
          <div class="sec s2">2 · Duvar</div>
          <div class="g2"><div class="field"><label>Duvar Metretül Fiyatı</label>
            <div class="suffix"><MoneyInput v-model="gider.duvar" class="inp" /><span>₺</span></div></div></div>
          <div class="sec s3">3 · Raf</div>
          <div class="g2"><div class="field"><label>Raf Metretül Fiyatı</label>
            <div class="suffix"><MoneyInput v-model="gider.raf" class="inp" /><span>₺</span></div></div></div>
          <div class="sec s4">4 · Masa</div>
          <div class="g2"><div class="field"><label>Masa Fiyatı</label>
            <div class="suffix"><MoneyInput v-model="gider.masa" class="inp" /><span>₺</span></div></div></div>
        </div>
        <footer><button class="btn btn-dark" @click="giderOpen = false">Tamam</button></footer>
      </div>
    </div>

    <div v-if="toastMsg" class="toast">{{ toastMsg }}</div>
  </div>
</template>
