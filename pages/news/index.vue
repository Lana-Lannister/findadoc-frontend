<template>
  <div>
    <v-container>
      <div id="all-news">
        <h1>{{ $t("news.important") }}</h1>
        <div v-for="article of articles" :key="article.slug">
          <NuxtLink
            :to="
              localePath({ name: 'news-slug', params: { slug: article.slug } })
            "
          >
            <img :src="article.img" />
            <div>
              <h2>{{ article.title }}</h2>
              <p>{{ $t("news.by", [article.author.name]) }}</p>
              <p>{{ article.description }}</p>
            </div>
            <v-btn text color="accent">{{ $t("general.readMore") }}</v-btn>
          </NuxtLink>
        </div>
      </div>
    </v-container>
  </div>
</template>

<script>
export default {
  async asyncData({ $content }) {
    const articles = await $content("articles")
      .only(["title", "description", "img", "slug", "author"])
      .sortBy("createdAt", "asc")
      .fetch();

    return {
      articles,
    };
  },
};
</script>

<style scoped>
#all-news {
  padding: 40px;
}
</style>
