<template>
  <b-container>
    <b-card no-body class="mb-3">
      <b-tabs card @input="refreshTOP" v-model="tabIndex">
        <b-tab title-link-class="no-tab" >
          <div @click="tabIndex = 0" slot="title" style="margin:-6px 0 -5px 0">
            <nuxt-link :to="'/space/' + parentSpace" v-if="!isGlobal" title="Go up" v-b-tooltip.hover>
              <icon name="long-arrow-alt-up" scale="2.5" class="go-up-arrow" style="margin-top:-8px" />
            </nuxt-link>
            <b-img rounded="circle" :src="'/images/users/small/' + space + '.jpg'" class="user-image"/>
            <strong class="ml-1">{{header}}</strong>
          </div>
          <space-terminal :path="spaceId" />
          <space-view :path="spaceId" />
        </b-tab>
        <b-tab title="Private Chat" v-if="!isGlobal">
          <template v-if="!isOwnSpace">
            <space-terminal :path="subspace(`16.${$auth.$state.user.id}`)" />
            <space-view :path="subspace(`16.${$auth.$state.user.id}`)" />
          </template>
          <b-tabs vertical pills v-else>
            <b-tab v-for="child in children" :key="child.id" :title="child.user">
            <space-terminal :path="subspace(`16.${child.id}`)" />
            <space-view :path="subspace(`16.${child.id}`)" />
            </b-tab>
          </b-tabs>
        </b-tab>
        <b-tab title="TOP" v-if="isOwnSpace">
          <processes-view :path="spaceId" ref="pv" />
        </b-tab>
        <b-tab title="Poll">
          <poll-view :path="spaceId" :isOwner="isOwnSpace" />
        </b-tab>
        <b-tab title="Notifications" v-if="isOwnSpace"> 
          <space-view :path="subspace(12)" />
        </b-tab>
        <template slot="tabs">
          <space-explorer :path="spaceId" :spaces="children" />
        </template>
      </b-tabs>
    </b-card>
    <SpaceSlider :spaces="children" v-if="!isGlobal" />
  </b-container>
</template>

<script>
import SpaceView from '~/components/SpaceView.vue'
import SpaceTerminal from '~/components/SpaceTerminal.vue'
import ProcessesView from '~/components/ProcessesView.vue'
import SpaceExplorer from '~/components/SpaceExplorer.vue'
import SpaceSlider from '~/components/SpaceSlider.vue'
import PollView from '~/components/PollView.vue'

import 'vue-awesome/icons/long-arrow-alt-up'

export default {
  middleware: ['auth', 'resolvePath'],
  asyncData: ctx => ctx.app.$axios.$get(`meta/space/${ctx.params.space}`),
  data () {
    return {
      tabIndex: 0
    }
  },
  computed: {
    header () {
      return this.space + (this.isGlobal ? '' : "'s") + ' space'
    },
    parentSpace () {
      if (`${this.spaceId}`.indexOf('.') < 0) return 'global'
      return this.spaceId.splice(this.spaceId.lastIndexOf('.'))
    }
  },
  components: {
    SpaceView, SpaceTerminal, ProcessesView, SpaceExplorer, SpaceSlider, PollView
  },
  methods: {
    refreshTOP (index) {
      if (this.isOwnSpace && index === 2) this.$refs.pv.refresh()
    },
    subspace (id) {
      return this.isGlobal ? id : `${this.spaceId}.${id}`
    }
  }
}
</script>

<style>
.profile-image {
  width:15vw;
  height:15vw;
  position:absolute;
  bottom:10px;
  right:30px;
  border: 4px ridge white;
  display: inline-block;
}

.profile-header {
  background-color: lightgray;
  font-size: 4.5vw;
  line-height: 100%;
  position:relative;
}

.user-image {
  margin-top: -8px;
}

.no-tab {
  background: initial !important;
  border: initial !important;
}

.go-up-arrow {
  margin-bottom: -10px
}
</style>
