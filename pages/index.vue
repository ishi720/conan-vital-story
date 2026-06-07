<template>
  <div class="wrap">
    <h1 class="app-title">名探偵コナン 進捗DB</h1>

    <div class="filters">
      <span class="filter-label">フィルター</span>
      <button
        v-for="f in filterOptions"
        :key="f.key"
        :class="['filter-btn', activeFilter === f.key && `act-${f.key}`]"
        @click="activeFilter = f.key"
      >{{ f.label }}</button>
      <div class="search-wrap">
        <span class="search-icon">🔍</span>
        <input v-model="query" type="text" placeholder="タイトル検索..." />
      </div>
    </div>

    <table>
      <colgroup>
        <col style="width:70px" />
        <col />
        <col style="width:90px" />
        <col style="width:80px" />
        <col style="width:180px" />
      </colgroup>
      <thead>
        <tr>
          <th @click="toggleSort('episode')">話数 {{ sortIcon('episode') }}</th>
          <th>タイトル</th>
          <th @click="toggleSort('volume')">単行本 {{ sortIcon('volume') }}</th>
          <th>進捗</th>
          <th>カテゴリ</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="ep in sorted" :key="ep.id">
          <td class="ep">{{ ep.episode }}</td>
          <td class="ttl">{{ ep.title }}</td>
          <td class="vol">第{{ ep.volume }}巻</td>
          <td>
            <span v-if="hasProgress(ep)" class="flag-yes">あり</span>
            <span v-else class="flag-no">なし</span>
          </td>
          <td>
            <div class="cats">
              <template v-if="ep.progress">
                <span v-if="ep.categories.includes('black_org')" class="badge b-org">黒の組織</span>
                <span v-if="ep.categories.includes('new_character')" class="badge b-char">キャラ追加</span>
                <span v-if="ep.categories.length === 0" class="badge b-other">その他</span>
              </template>
              <span v-else class="no-cat">-</span>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <p v-if="sorted.length === 0" class="empty">該当するエピソードが見つかりませんでした</p>
  </div>
</template>

<script setup lang="ts">
import episodesRaw from '~/data/episodes.json'

interface Episode {
  id: number
  episode: number
  title: string
  volume: number
  progress: boolean
  categories: string[]
}

const episodes = episodesRaw as Episode[]

const activeFilter = ref<'all' | 'yes' | 'no'>('all')
const query = ref('')
const sortKey = ref<'episode' | 'volume'>('episode')
const sortDir = ref(1)

const filterOptions = [
  { key: 'all', label: 'すべて' },
  { key: 'yes', label: '進捗あり' },
  { key: 'no',  label: '進捗なし' },
]

const hasProgress = (ep: Episode) => ep.progress

const filtered = computed(() => {
  let list = episodes
  if (activeFilter.value === 'yes') list = list.filter(hasProgress)
  if (activeFilter.value === 'no')  list = list.filter(e => !hasProgress(e))
  if (query.value.trim()) {
    const q = query.value.toLowerCase()
    list = list.filter(e => e.title.toLowerCase().includes(q))
  }
  return list
})

const sorted = computed(() => {
  return [...filtered.value].sort((a, b) => {
    const av = a[sortKey.value] ?? 9999
    const bv = b[sortKey.value] ?? 9999
    return (av > bv ? 1 : av < bv ? -1 : 0) * sortDir.value
  })
})

function toggleSort(key: 'episode' | 'volume') {
  if (sortKey.value === key) sortDir.value *= -1
  else { sortKey.value = key; sortDir.value = 1 }
}

function sortIcon(key: string) {
  if (sortKey.value !== key) return '↕'
  return sortDir.value > 0 ? '↑' : '↓'
}
</script>

<style scoped>
.wrap {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
  font-family: sans-serif;
}
.app-title {
  font-size: 1.3rem;
  font-weight: 700;
  margin-bottom: 1.25rem;
  color: #111;
}
.filters {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}
.filter-label {
  font-size: 11px;
  color: #888;
}
.filter-btn {
  font-size: 12px;
  padding: 4px 12px;
  border-radius: 6px;
  border: 1px solid #ddd;
  background: transparent;
  color: #555;
  cursor: pointer;
}
.filter-btn:hover { background: #f5f5f5; }
.act-all { background: #f0f0f0; border-color: #aaa; color: #111; font-weight: 500; }
.act-yes { background: #e8f5e9; border-color: #66bb6a; color: #2e7d32; font-weight: 500; }
.act-no  { background: #f5f5f5; border-color: #bbb; color: #555; font-weight: 500; }
.search-wrap {
  position: relative;
  margin-left: auto;
}
.search-icon {
  position: absolute;
  left: 8px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 12px;
}
.search-wrap input {
  padding: 4px 8px 4px 26px;
  font-size: 13px;
  border: 1px solid #ddd;
  border-radius: 6px;
  outline: none;
  width: 180px;
}
table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
  table-layout: fixed;
}
th {
  font-size: 11px;
  color: #888;
  font-weight: 500;
  padding: 6px 10px;
  border-bottom: 1px solid #eee;
  text-align: left;
  cursor: pointer;
  user-select: none;
  white-space: nowrap;
}
th:hover { color: #333; }
td {
  padding: 9px 10px;
  border-bottom: 1px solid #f0f0f0;
  vertical-align: middle;
}
tr:hover td { background: #fafafa; }
.ep { font-weight: 600; color: #111; }
.ttl { font-weight: 500; color: #111; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.vol { font-size: 12px; color: #777; }
.flag-yes { font-size: 11px; padding: 2px 8px; border-radius: 4px; background: #e8f5e9; color: #2e7d32; border: 1px solid #a5d6a7; }
.flag-no  { font-size: 12px; color: #ccc; }
.cats { display: flex; gap: 4px; flex-wrap: wrap; }
.badge { font-size: 11px; padding: 2px 8px; border-radius: 4px; border: 1px solid; white-space: nowrap; }
.b-org   { background: #fdecea; color: #c62828; border-color: #ef9a9a; }
.b-char  { background: #e3f2fd; color: #1565c0; border-color: #90caf9; }
.b-other { background: #f3f3f3; color: #777; border-color: #ccc; }
.no-cat { font-size: 12px; color: #ccc; }
.empty { text-align: center; padding: 3rem 0; color: #aaa; font-size: 13px; }
</style>
