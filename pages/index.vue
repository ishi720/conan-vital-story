<template>
  <div class="wrap">
    <h1 class="app-title">名探偵コナン 進捗DB</h1>

    <div class="filters">
      <span class="filter-label">フィルター</span>
      <button
        v-for="f in filterOptions"
        :key="f.key"
        :class="['filter-btn', activeFilter === f.key && `act-${f.key}`]"
        @click="activeFilter = f.key as 'all' | 'yes' | 'black_org' | 'new_character'"
      >{{ f.label }}</button>
    </div>

    <table>
      <colgroup>
        <col style="width:220px" />
        <col style="width:90px" />
        <col />
        <col style="width:80px" />
        <col style="width:180px" />
      </colgroup>
      <thead>
        <tr>
          <th @click="toggleSort('episode')">エピソード {{ sortIcon('episode') }}</th>
          <th @click="toggleSort('volume')">単行本 {{ sortIcon('volume') }}</th>
          <th>ファイル</th>
          <th>進捗</th>
          <th>カテゴリ</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(ep, index) in sorted" :key="ep.id">
          <td v-if="isFirstInGroup(index)" class="ep" :rowspan="groupSize(index)"><span class="ep-num">{{ ep.episode }}</span>{{ ep.title }}</td>
          <td class="vol">第{{ ep.volume }}巻</td>
          <td class="file">{{ ep.file }}</td>
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
  file: string
}

const episodes = episodesRaw as Episode[]

const activeFilter = ref<'all' | 'yes' | 'black_org' | 'new_character'>('all')
const sortKey = ref<'episode' | 'volume'>('episode')
const sortDir = ref(1)

const filterOptions = [
  { key: 'all',           label: 'すべて' },
  { key: 'yes',           label: '進捗有' },
  { key: 'black_org',     label: '黒の組織' },
  { key: 'new_character', label: 'キャラ追加' },
]

const hasProgress = (ep: Episode) => ep.progress

const filtered = computed(() => {
  let list = episodes
  if (activeFilter.value === 'yes') list = list.filter(hasProgress)
  else if (activeFilter.value === 'black_org') list = list.filter(e => e.categories.includes('black_org'))
  else if (activeFilter.value === 'new_character') list = list.filter(e => e.categories.includes('new_character'))
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

function isFirstInGroup(index: number): boolean {
  if (index === 0) return true
  return sorted.value[index].episode !== sorted.value[index - 1].episode
}

function groupSize(index: number): number {
  const ep = sorted.value[index].episode
  let count = 0
  for (let i = index; i < sorted.value.length && sorted.value[i].episode === ep; i++) count++
  return count
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
.act-all           { background: #f0f0f0; border-color: #aaa;    color: #111;    font-weight: 500; }
.act-yes           { background: #e8f5e9; border-color: #66bb6a; color: #2e7d32; font-weight: 500; }
.act-black_org     { background: #fdecea; border-color: #ef9a9a; color: #c62828; font-weight: 500; }
.act-new_character { background: #e3f2fd; border-color: #90caf9; color: #1565c0; font-weight: 500; }
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
.ep { font-weight: 500; color: #111; vertical-align: top; }
.ep-num { font-size: 11px; color: #aaa; margin-right: 6px; }
.ttl { font-weight: 500; color: #111; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.file { font-size: 12px; color: #555; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
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
