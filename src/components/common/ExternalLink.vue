<template>
  <div>
    <a v-on:click="togglePopup"><b>{{text}}</b></a>
    <div :class="`modal ${active ? 'is-active' : ''}`">
      <div class="modal-background"></div>
      <div class="modal-card">
        <section class="modal-card-body">
          <h2 class="label">{{$t('global.external_link_title')}}</h2>
          <p>{{$t('global.external_link_text')}}</p>
          <hr class="divider">
          <button class="button is-success" v-on:click="openLink">{{$t('global.continue')}}</button>
          &nbsp;
          <button class="button is-danger" v-on:click="togglePopup">{{$t('global.nevermind')}}</button>
        </section>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ExternalLink',
  props: ['link', 'text'],
  data() {
    return {
      active: false,
    };
  },
  methods: {
    togglePopup() {
      this.active = !this.active;
    },
    /** @method
     * @name openLink
     * opens confirmation dialog to view external link
     */
    openLink() {
      this.togglePopup();
      window.open(this.link, '_blank');
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.modal-card-body {
  font-weight: normal !important;
  font-size: 12pt;
}
.is-normal {
  background: #00d0a1;
  color: #fff;
  border: #00deac;
}
.is-normal:hover {
  color: #fff;
  background: #00deac;
}
.modal-card-body {
  width: 490px;
  margin: 0 auto;
  text-align: center;
  padding: 2rem;
}
.modal-card {
  border-radius: 5px;
  margin-top: -10%;
}
.modal-card-head {
  background: #fff;
  border-radius: none !important;
}
.modal-card-body {
  background: #e7ebee;
}
.chat-group {
  position: absolute;
  bottom: 2rem;
  right: 0;
  left: 0;
  height: 5rem;
  padding: 1rem;
}
.input-collection {
  background: #e7ebee;
  border-radius: 5px;
  width: 100%;
  height: 100%;
}
.input-collection i {
  color: #666;
  cursor: pointer;
}
.send {
  position: absolute;
  right: 2.2rem;
  top: 1.75rem;
  font-size: 19pt;
}
.upload {
  position: absolute;
  left: 2.2rem;
  top: 1.75rem;
  font-size: 19pt;
}
.fingerprint {
  position: absolute;
  right: 4.75rem;
  top: 1.75rem;
  font-size: 19pt;
}
.messageuser {
  margin-left: 3rem;
  margin-top: 0.25rem;
  background: transparent;
  border: none;
  box-shadow: none;
  font-size: 12pt;
  width: calc(100% - 8rem);
}
input:focus, input:focus{
  outline: none !important;
  -webkit-box-shadow: none !important;
  box-shadow: none !important;
  border: none !important;
}
input::placeholder {
  color: #666;
  opacity: 0.75;
}
.address {
  font-size: 10pt;
  color: #ccc;
}
.modal-card-title {
  margin-left: 1rem;
}
.circle-icon {
  height: 85px;
  width: 85px;
  margin-bottom: 1rem;
}
.badges, .padded {
  padding: 0 0.5rem;
}

.person {
  font-size: 18pt;
}
.label {
  font-size: 14pt;
}
</style>
