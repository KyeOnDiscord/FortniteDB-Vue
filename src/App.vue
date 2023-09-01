<template>
  <table
    class="table table-hover table-sm table-bordered border-dark-subtle"
    style="max-width: 100%"
    v-if="skins"
  >
    <thead style="max-width: 100%">
      <tr>
        <th scope="col">#</th>
        <th scope="col">Icon</th>
        <th scope="col">ID & Path</th>
        <th scope="col">Name</th>
        <th scope="col">Length</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(skin, index) in skins" :key="index">
        <th scope="row">{{ index + 1 }}</th>
        <td>
          <img
            :src="skin.images.icon ?? null"
            style="width: 100px; height: auto"
          />
        </td>
        <td style="background-color: #1f1f1f">
          <code>
            {{ skin.id }}
            <br />
            <br />
            {{ skin.path }}
          </code>
        </td>
        <td>
          {{ skin.name }}
        </td>
        <td>{{ skin.name.length }}</td>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  async mounted() {
    const resp = await fetch("https://fortnite-api.com/v2/cosmetics/br");
    let json = await resp.json();
    // console.log(json);
    this.skins = json.data
      .filter((item) => item.type.backendValue === "AthenaCharacter")
      .sort((a, b) => (a.id > b.id ? 1 : -1));
    console.log(this.skins);
  },
  data() {
    return {
      skins: null,
    };
  },
};
</script>

<style scoped>
code {
  padding: 2px 4px;
  border-radius: 4px;
  font-size: 12px;
  font-family: "Menlo";
  color: #569cd6;
  background-color: #1f1f1f;
}
</style>
