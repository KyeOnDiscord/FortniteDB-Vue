<template>
  <div class="container d-flex flex-row mb-3">
    <div class="dropdown me-3">
      <button
        class="btn btn-secondary dropdown-toggle"
        type="button"
        data-bs-toggle="dropdown"
        aria-expanded="false"
      >
        Cosmetic Type
      </button>
      <ul class="dropdown-menu">
        <li v-for="cosmeticType in CosmeticTypes">
          <a
            class="dropdown-item"
            @click="this.selectedCosmeticType = cosmeticType"
            >{{ cosmeticType }}</a
          >
        </li>
      </ul>
    </div>
    <div v-for="key in Object.keys(columns)" class="form-check me-3">
      <input class="form-check-input" type="checkbox" v-model="columns[key]" />
      <label class="form-check-label">
        {{ key }}
      </label>
    </div>
  </div>

  <table
    class="table table-hover table-sm table-bordered border-dark-subtle"
    style="max-width: 100%"
    v-if="this.CosmeticsList"
  >
    <thead style="max-width: 100%">
      <tr>
        <th
          v-for="key in Object.keys(columns).filter((x) => columns[x] == true)"
          :key="key"
          scope="col"
        >
          {{ key }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="(skin, index) in this.CosmeticsList.filter(
          (item) => item.type.backendValue === this.selectedCosmeticType
        ).sort((a, b) => (a.id > b.id ? 1 : -1))"
        :key="index"
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
          <code
            ><a
              :href="
                'https://fortnitecentral.genxgames.gg/api/v1/export?path=' +
                skin.path
              "
              target="_blank"
              style="text-decoration: none"
              >{{ skin.id }}</a
            ></code
          >
        </td>
        <td v-if="columns['Path']">
          <code
            ><a
              :href="
                'https://fortnitecentral.genxgames.gg/api/v1/export?path=' +
                skin.path
              "
              target="_blank"
              style="text-decoration: none"
              >{{ skin.path }}</a
            ></code
          >
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
    </tbody>
  </table>
</template>

<script>
export default {
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
