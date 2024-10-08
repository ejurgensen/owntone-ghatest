<template>
  <transition name="fade">
    <div v-if="show" class="modal is-active">
      <div class="modal-background" @click="$emit('close')" />
      <div class="modal-content">
        <div class="card">
          <div class="card-content">
            <p class="title is-4" v-text="item.title" />
            <p class="subtitle" v-text="item.artist" />
            <div class="content is-small">
              <p v-if="item.album">
                <span class="heading" v-text="$t('dialog.queue-item.album')" />
                <a
                  class="title is-6 has-text-link"
                  @click="open_album"
                  v-text="item.album"
                />
              </p>
              <p v-if="item.album_artist">
                <span
                  class="heading"
                  v-text="$t('dialog.queue-item.album-artist')"
                />
                <a
                  class="title is-6 has-text-link"
                  @click="open_album_artist"
                  v-text="item.album_artist"
                />
              </p>
              <p v-if="item.composer">
                <span
                  class="heading"
                  v-text="$t('dialog.queue-item.composer')"
                />
                <span class="title is-6" v-text="item.composer" />
              </p>
              <p v-if="item.year">
                <span class="heading" v-text="$t('dialog.queue-item.year')" />
                <span class="title is-6" v-text="item.year" />
              </p>
              <p v-if="item.genre">
                <span class="heading" v-text="$t('dialog.queue-item.genre')" />
                <a
                  class="title is-6 has-text-link"
                  @click="open_genre"
                  v-text="item.genre"
                />
              </p>
              <p v-if="item.disc_number">
                <span
                  class="heading"
                  v-text="$t('dialog.queue-item.position')"
                />
                <span
                  class="title is-6"
                  v-text="[item.disc_number, item.track_number].join(' / ')"
                />
              </p>
              <p v-if="item.length_ms">
                <span
                  class="heading"
                  v-text="$t('dialog.queue-item.duration')"
                />
                <span
                  class="title is-6"
                  v-text="$filters.durationInHours(item.length_ms)"
                />
              </p>
              <p>
                <span class="heading" v-text="$t('dialog.queue-item.path')" />
                <span class="title is-6" v-text="item.path" />
              </p>
              <p>
                <span class="heading" v-text="$t('dialog.queue-item.type')" />
                <span class="title is-6">
                  <span
                    v-text="
                      `${$t(`media.kind.${item.media_kind}`)} - ${$t(`data.kind.${item.data_kind}`)}`
                    "
                  />
                </span>
              </p>
              <p v-if="item.samplerate">
                <span
                  class="heading"
                  v-text="$t('dialog.queue-item.quality')"
                />
                <span class="title is-6">
                  <span v-text="item.type" />
                  <span
                    v-if="item.samplerate"
                    v-text="
                      $t('dialog.queue-item.samplerate', {
                        rate: item.samplerate
                      })
                    "
                  />
                  <span
                    v-if="item.channels"
                    v-text="
                      $t('dialog.queue-item.channels', {
                        channels: $filters.channels(item.channels)
                      })
                    "
                  />
                  <span
                    v-if="item.bitrate"
                    v-text="
                      $t('dialog.queue-item.bitrate', { rate: item.bitrate })
                    "
                  />
                </span>
              </p>
            </div>
          </div>
          <footer class="card-footer">
            <a class="card-footer-item has-text-dark" @click="remove">
              <mdicon class="icon" name="delete" size="16" />
              <span class="is-size-7" v-text="$t('dialog.queue-item.remove')" />
            </a>
            <a class="card-footer-item has-text-dark" @click="play">
              <mdicon class="icon" name="play" size="16" />
              <span class="is-size-7" v-text="$t('dialog.queue-item.play')" />
            </a>
          </footer>
        </div>
      </div>
      <button
        class="modal-close is-large"
        aria-label="close"
        @click="$emit('close')"
      />
    </div>
  </transition>
</template>

<script>
import SpotifyWebApi from 'spotify-web-api-js'
import { useServicesStore } from '@/stores/services'
import webapi from '@/webapi'

export default {
  name: 'ModalDialogQueueItem',
  props: { item: { required: true, type: Object }, show: Boolean },
  emits: ['close'],

  setup() {
    return { servicesStore: useServicesStore() }
  },

  data() {
    return {
      spotify_track: {}
    }
  },

  watch: {
    item() {
      if (this.item?.data_kind === 'spotify') {
        const spotifyApi = new SpotifyWebApi()
        spotifyApi.setAccessToken(this.servicesStore.spotify.webapi_token)
        spotifyApi
          .getTrack(this.item.path.slice(this.item.path.lastIndexOf(':') + 1))
          .then((response) => {
            this.spotify_track = response
          })
      } else {
        this.spotify_track = {}
      }
    }
  },

  methods: {
    open_album() {
      if (this.item.data_kind === 'spotify') {
        this.$router.push({
          name: 'music-spotify-album',
          params: { id: this.spotify_track.album.id }
        })
      } else if (this.item.media_kind === 'podcast') {
        this.$router.push({
          name: 'podcast',
          params: { id: this.item.album_id }
        })
      } else if (this.item.media_kind === 'audiobook') {
        this.$router.push({
          name: 'audiobooks-album',
          params: { id: this.item.album_id }
        })
      } else if (this.item.media_kind === 'music') {
        this.$router.push({
          name: 'music-album',
          params: { id: this.item.album_id }
        })
      }
    },
    open_album_artist() {
      if (this.item.data_kind === 'spotify') {
        this.$router.push({
          name: 'music-spotify-artist',
          params: { id: this.spotify_track.artists[0].id }
        })
      } else if (
        this.item.media_kind === 'music' ||
        this.item.media_kind === 'podcast'
      ) {
        this.$router.push({
          name: 'music-artist',
          params: { id: this.item.album_artist_id }
        })
      } else if (this.item.media_kind === 'audiobook') {
        this.$router.push({
          name: 'audiobooks-artist',
          params: { id: this.item.album_artist_id }
        })
      }
    },
    open_genre() {
      this.$router.push({
        name: 'genre-albums',
        params: { name: this.item.genre },
        query: { media_kind: this.item.media_kind }
      })
    },
    play() {
      this.$emit('close')
      webapi.player_play({ item_id: this.item.id })
    },
    remove() {
      this.$emit('close')
      webapi.queue_remove(this.item.id)
    }
  }
}
</script>

<style></style>
