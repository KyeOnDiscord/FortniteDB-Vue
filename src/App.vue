<template>
  <section class="container filter-panel mb-3">
    <div
      class="d-flex flex-wrap align-items-start justify-content-between gap-2 mb-3"
    >
      <div>
        <h1 class="h5 mb-1">Cosmetic filters</h1>
        <p class="text-body-secondary mb-0">
          Search cosmetics and customize the table.
        </p>
      </div>
      <span class="badge rounded-pill text-bg-primary">
        {{ filteredCosmetics.length }} results
      </span>
    </div>

    <div class="row g-3">
      <div class="col-12 col-md-4">
        <label for="cosmetic-type" class="form-label fw-semibold">
          Cosmetic type
        </label>
        <select
          id="cosmetic-type"
          v-model="selectedCosmeticType"
          class="form-select"
        >
          <option
            v-for="cosmeticType in CosmeticTypes"
            :key="cosmeticType"
            :value="cosmeticType"
          >
            {{ cosmeticType }}
          </option>
        </select>
      </div>
      <div class="col-12 col-md-8">
        <label for="cosmetic-search" class="form-label fw-semibold">
          Search by name or ID
        </label>
        <div class="input-group">
          <input
            id="cosmetic-search"
            v-model="searchQuery"
            type="search"
            class="form-control"
            placeholder="Enter a cosmetic name or ID"
            autocomplete="off"
          />
          <button
            v-if="searchQuery"
            class="btn btn-outline-secondary"
            type="button"
            @click="searchQuery = ''"
          >
            Clear
          </button>
        </div>
      </div>
    </div>

    <fieldset class="column-filters mt-3">
      <legend class="form-label fw-semibold mb-2">Visible columns</legend>
      <div class="d-flex flex-wrap gap-2">
        <label
          v-for="key in Object.keys(columns)"
          :key="key"
          class="column-toggle"
          :class="{ active: columns[key] }"
        >
          <input
            v-model="columns[key]"
            class="form-check-input"
            type="checkbox"
          />
          <span>{{ key }}</span>
        </label>
      </div>
    </fieldset>
  </section>

  <table
    class="table table-hover table-sm table-bordered border-dark-subtle"
    style="max-width: 100%"
    v-if="this.CosmeticsList"
  >
    <thead style="max-width: 100%">
      <tr>
        <th
          v-for="key in visibleColumns"
          :key="key"
          scope="col"
        >
          {{ key }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="(skin, index) in filteredCosmetics"
        :key="skin.id"
      >
        <th v-if="columns['#']" scope="row">{{ index + 1 }}</th>
        <td v-if="columns['Icon']">
          <img
            :src="skin.images.smallIcon ?? null"
            :class="skin.rarity.value"
            loading="lazy"
          />
        </td>
        <td v-if="columns['ID']" style="max-width: 20%">
          <code>
            <button
              type="button"
              class="copy-button"
              @click="copyToClipboard(skin.id)"
            >
              {{ skin.id }}
            </button>
          </code>
        </td>
        <td v-if="columns['Path']">
          <code>
            <button
              type="button"
              class="copy-button"
              @click="copyToClipboard(skin.path)"
            >
              {{ skin.path }}
            </button>
          </code>
        </td>
        <td v-if="columns['Name']">
          {{ skin.name }}
        </td>
        <td v-if="columns['Description']" style="max-width: 20%">
          {{ skin.description }}
        </td>
        <td v-if="columns['Set']">
          {{ skin.set == null ? "None" : skin.set.value }}
        </td>
        <td v-if="columns['Introduction']">
          {{ skin.introduction == null ? "None" : skin.introduction.text }}
        </td>
      </tr>
      <tr v-if="filteredCosmetics.length === 0">
        <td
          :colspan="Math.max(visibleColumns.length, 1)"
          class="text-center text-body-secondary py-4"
        >
          No cosmetics match your filters.
        </td>
      </tr>
    </tbody>
  </table>

  <div
    v-if="copyFeedback"
    class="copy-toast"
    :class="
      copyFeedback === 'Copied!' ? 'copy-toast-success' : 'copy-toast-error'
    "
    role="status"
    aria-live="polite"
  >
    {{ copyFeedback }}
  </div>
</template>

<script>
export default {
  computed: {
    visibleColumns() {
      return Object.keys(this.columns).filter((key) => this.columns[key]);
    },
    filteredCosmetics() {
      const searchQuery = this.searchQuery.trim().toLowerCase();

      return (this.CosmeticsList ?? [])
        .filter((item) => {
          const matchesType =
            item.type.backendValue === this.selectedCosmeticType;
          const matchesSearch =
            !searchQuery ||
            item.name.toLowerCase().includes(searchQuery) ||
            item.id.toLowerCase().includes(searchQuery);

          return matchesType && matchesSearch;
        })
        .sort((a, b) => new Date(b.added) - new Date(a.added));
    },
  },
  beforeUnmount() {
    clearTimeout(this.copyFeedbackTimeout);
  },
  async mounted() {
    const resp = await fetch(
      "https://fortnite-api.com/v2/cosmetics/br?responseFlags=1"
    );
    let json = await resp.json();
    this.CosmeticTypes = this.getCosmeticBackends(json.data);
    this.CosmeticsList = json.data;
  },
  data() {
    return {
      CosmeticsList: null,
      CosmeticTypes: null,
      selectedCosmeticType: "AthenaCharacter",
      searchQuery: "",
      copyFeedback: "",
      copyFeedbackTimeout: null,
      columns: {
        "#": true,
        Icon: true,
        ID: true,
        Path: false,
        Name: true,
        Description: true,
        Set: false,
        Introduction: false,
      },
    };
  },
  methods: {
    async copyToClipboard(value) {
      try {
        await navigator.clipboard.writeText(value);
        this.copyFeedback = "Copied!";
      } catch (error) {
        console.error("Failed to copy to clipboard:", error);
        this.copyFeedback = "Copy failed";
      }

      clearTimeout(this.copyFeedbackTimeout);
      this.copyFeedbackTimeout = setTimeout(() => {
        this.copyFeedback = "";
        this.copyFeedbackTimeout = null;
      }, 1500);
    },
    getCosmeticBackends(CosmeticsList) {
      let nonDistinctList = CosmeticsList.map((item) => item.type.backendValue);
      return [...new Set(nonDistinctList)]; // Spread syntax to create a new array
    },
  },
};
</script>

<style scoped>
code {
  padding: 2px 4px;
  border-radius: 4px;
  font-size: 14px;
  font-family: "Courier New", Courier, monospace;
  color: #569cd6;
  background-color: #1f1f1f;
}

.copy-button {
  padding: 0;
  border: 0;
  background: none;
  color: inherit;
  font: inherit;
  text-align: left;
  cursor: pointer;
}

.filter-panel {
  padding: 1.25rem;
  border: 1px solid var(--bs-border-color);
  border-radius: 0.75rem;
  background-color: var(--bs-tertiary-bg);
}

.column-filters {
  min-width: 0;
  padding: 0;
  border: 0;
  margin: 0;
}

.column-toggle {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  padding: 0.4rem 0.7rem;
  border: 1px solid var(--bs-border-color);
  border-radius: 50rem;
  background-color: var(--bs-body-bg);
  cursor: pointer;
  transition:
    border-color 0.15s ease-in-out,
    background-color 0.15s ease-in-out;
}

.column-toggle.active {
  border-color: var(--bs-primary);
  background-color: rgba(var(--bs-primary-rgb), 0.08);
}

.column-toggle .form-check-input {
  margin: 0;
  cursor: pointer;
}

.copy-toast {
  position: fixed;
  right: 1.5rem;
  bottom: 1.5rem;
  z-index: 1050;
  padding: 0.75rem 1rem;
  border-radius: 0.375rem;
  color: #fff;
  box-shadow: 0 0.25rem 0.75rem rgb(0 0 0 / 25%);
}

.copy-toast-success {
  background-color: #198754;
}

.copy-toast-error {
  background-color: #dc3545;
}

img {
  width: 100px;
  height: auto;
}

.uncommon {
  background: -o-radial-gradient(#6abb1e, #175117);
  background: radial-gradient(#6abb1e, #175117);
  border: 2px solid #88e339;
}
.rare {
  background: -o-radial-gradient(#2cc0ff, #143877);
  background: radial-gradient(#2cc0ff, #143877);
  border: 2px solid #37d0ff;
}
.epic {
  background: -o-radial-gradient(#c359ff, #4c2483);
  background: radial-gradient(#c359ff, #4c2483);
  border: 2px solid #ea5eff;
}
.legendary {
  background: -o-radial-gradient(#ea8d23, #78371d);
  background: radial-gradient(#ea8d23, #78371d);
  border: 2px solid #e98d4b;
}
.mythic {
  background: -o-radial-gradient(#f5cb44, #ea8d23);
  background: radial-gradient(#f5cb44, #ea8d23);
  border: 2px solid #e98d4b;
}
.dark {
  background: -o-radial-gradient(#fb22df, #520c6f);
  background: radial-gradient(#fb22df, #520c6f);
  border: 2px solid #ff42e7;
}
.frozen {
  background: -o-radial-gradient(#94dfff, #269ed6);
  background: radial-gradient(#94dfff, #269ed6);
  border: 2px solid #c4dff7;
}
.icon {
  background: -o-radial-gradient(#36b7b7, #256b6b);
  background: radial-gradient(#36b7b7, #256b6b);
  border: 2px solid #52e0e0;
}
.lava {
  background: -o-radial-gradient(#ea8d23, #6a0a31);
  background: radial-gradient(#ea8d23, #6a0a31);
  border: 2px solid #d19635;
}
.marvel {
  background: -o-radial-gradient(#c53333, #761b1b);
  background: radial-gradient(#c53333, #761b1b);
  border: 2px solid #ef3537;
}
.shadow {
  background: -o-radial-gradient(#717171, #191919);
  background: radial-gradient(#717171, #191919);
  border: 2px solid #949494;
}
.gaminglegends {
  background: -o-radial-gradient(#5447d4, #312497);
  background: radial-gradient(#5447d4, #312497);
  border: 2px solid #8078ff;
}
.dc {
  background: -o-radial-gradient(#5475c7, #243461);
  background: radial-gradient(#5475c7, #243461);
  border: 2px solid #6094ce;
}
.slurp {
  background: -o-radial-gradient(#29f1a3, #12a9a4);
  background: radial-gradient(#29f1a3, #12a9a4);
  border: 2px solid #53f0ff;
}
.starwars {
  background: -o-radial-gradient(#1b366e, #081737);
  background: radial-gradient(#1b366e, #081737);
  border: 2px solid #e7c413;
}

.common {
  background: -o-radial-gradient(#bebebe, #646464);
  background: radial-gradient(#bebebe, #646464);
  border: 2px solid #b1b1b1;
}
</style>
