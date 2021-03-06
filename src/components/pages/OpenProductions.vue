<template>
  <div class="open-productions page">
    <div class="has-text-centered" v-if="isOpenProductionsLoading">
      <spinner />
    </div>
    <div v-else-if="openProductions.length > 0">
      <h1 class="title has-text-centered">
        <activity-icon />
        {{ $t('productions.home.title') }}
      </h1>
      <div
        :class="{
          'open-productions-list': true,
          'is-grid': openProductions && openProductions.length > 4
        }"
      >
        <div
          class="open-production has-text-centered"
          :key="production.id"
          v-for="production in openProductions">

          <router-link
            :to="getPath(production)"
          >
            <div
              class="avatar has-text-centered"
              v-bind:style="{
                background: getAvatarColor(production)
              }"
            >
              <span v-if="!production.has_avatar">
                {{ generateAvatar(production) }}
              </span>
              <img :src="getThumbnailPath(production)" v-else />
            </div>
            <div class="production-name">
              {{ production.name }}
            </div>
          </router-link>
          <router-link
            class="secondary"
            :to="assetsPath(production)"
          >
            {{ $t('assets.title') }}
          </router-link>
          -
          <router-link
            class="secondary"
            :to="shotsPath(production)"
          >
            {{ $t('shots.title') }}
          </router-link>
        </div>
      </div>
      <div class="has-text-centered new-production-link">
        <a
          @click="showNewModal"
        >
          {{ $t('productions.home.create_new') }}
        </a>
      </div>
    </div>

    <div class="has-text-centered welcome" v-else>
      <p class="kitsu-logo info">
        <img src="../../assets/illustrations/studio_collaboration.png" />
      </p>
      <h1 class="title has-text-centered">
        {{ $t('productions.home.welcome') }}
      </h1>
      <div v-if="isCurrentUserManager">
        <p class="has-text-centered info">
          {{ $t('productions.home.empty') }}
        </p>
        <p class="has-text-centered">
          <button-simple
            class="level-item big-button"
            :text="$t('productions.home.create_new')"
            :is-responsive="false"
            @click="showNewModal"
          />
        </p>
      </div>
      <div v-else-if="isCurrentUserClient">
        <p class="has-text-centered">
          {{ $t('productions.home.no_prod_for_client') }}
        </p>
      </div>
      <div v-else>
        <p class="has-text-centered">
          {{ $t('productions.home.no_task') }}
        </p>
      </div>
    </div>

    <edit-production-modal
      :active="modals.isNewDisplayed"
      :is-loading="editProduction.isLoading"
      :is-error="editProduction.isError"
      @confirm="confirmEditProduction"
      @cancel="hideNewModal"
    />

  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import { ActivityIcon } from 'vue-feather-icons'

import colors from '../../lib/colors.js'
import ButtonSimple from '../widgets/ButtonSimple'
import EditProductionModal from '../modals/EditProductionModal'
import Spinner from '../widgets/Spinner'

export default {
  name: 'open-productions',

  components: {
    ActivityIcon,
    ButtonSimple,
    EditProductionModal,
    Spinner
  },

  data () {
    return {
      modals: {
        isNewDisplayed: false
      }
    }
  },

  computed: {
    ...mapGetters([
      'editProduction',
      'isCurrentUserManager',
      'isCurrentUserClient',
      'isOpenProductionsLoading',
      'lastProductionScreen',
      'openProductions'
    ])
  },

  methods: {
    ...mapActions([
      'newProduction'
    ]),

    generateAvatar (production) {
      const firstLetter = production.name.length > 0 ? production.name[0] : 'P'
      return firstLetter.toUpperCase()
    },

    getAvatarColor (production) {
      return colors.fromString(production.name)
    },

    getPath (production) {
      return this.sectionPath(production, this.lastProductionScreen)
    },

    assetsPath (production) {
      return this.sectionPath(production, 'assets')
    },

    shotsPath (production) {
      return this.sectionPath(production, 'shots')
    },

    sequencesPath (production) {
      return this.sectionPath(production, 'sequences')
    },

    sectionPath (production, section) {
      let route = {
        name: section,
        params: {
          production_id: production.id
        }
      }
      if (production.production_type === 'tvshow') {
        route.name = `episode-${section}`
        route.params.episode_id = production.first_episode_id
      }
      return route
    },

    getThumbnailPath (production) {
      return `/api/pictures/thumbnails/projects/${production.id}.png`
    },

    confirmEditProduction (form) {
      this.newProduction({
        data: form,
        callback: (err, production) => {
          if (!err) {
            this.modals.isNewDisplayed = false
          }
        }
      })
    },

    showNewModal () {
      this.modals.isNewDisplayed = true
    },

    hideNewModal () {
      this.modals.isNewDisplayed = false
    }

  },

  watch: {},

  metaInfo () {
    return {
      title: `${this.$t('productions.home.title')} - Kitsu`
    }
  }
}
</script>

<style lang="scss" scoped>
h1 {
  margin-top: 3em;
  margin-bottom: 3em;
}

.is-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill,minmax(200px, 1fr));

  .open-production {
    margin: 1em auto 0 auto;
  }
}

.open-productions-list {
  max-width: 1000px;
  margin: auto;
  margin-top: 4em;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  text-align: center;

  .avatar {
    width: 100px;
    height: 100px;
    margin: auto;
    font-size: 64px;
    font-weight: bold;
    border-radius: 1em;
  }

  .avatar img {
    border-radius: 1em;
    width: 100%;
    height: 100%;
  }

  .open-production {
    overflow-wrap: break-word;
    padding: 10px;
    cursor: pointer;
    padding: 1em;
    flex: 1;
  }

  .open-production:hover .avatar{
    transform: scale(1.1);
    transition: transform .2s ease-in-out;
  }
}

.production-name {
  font-size: 1.4em;
  text-transform: uppercase;
  color: $grey;
  margin: 0.5em auto;
  width: 200px;
}

.welcome {
  max-width: 1000px;
  margin: auto;

  h1 {
    margin-top: 1em;
  }
}

.kitsu-logo {
  margin-top: 4em;
}

a.secondary {
  color: #BBB;
}

a.secondary:hover {
  text-decoration: underline
}

.new-production-link {
  margin-top: 4em;

  a {
    color: #BBB;
  }
}

@media screen and (max-width: 768px) {
  .title {
    margin-top: 1em;
  }

  .production-name {
    font-size: 1.1em;
  }
}
</style>
