<template src="./MiniPayment.html"></template>

<script>
import config from '@/config/config';
import CircleIcon from '@/components/common/CircleIcon';
import DwellerCachingHelper from '@/classes/DwellerCachingHelper.ts';
import MessageBroker from '@/classes/MessageBroker.ts';
import Ethereum from '@/classes/Ethereum';

const ethereum = new Ethereum('user-provided');

export default {
  name: 'MiniPayment',
  props: [
    'person',
    'toggle',
    'address',
  ],
  components: {
    CircleIcon,
  },
  data() {
    return {
      amount: null,
      icon: false,
      name: false,
      error: false,
      priceUsd: 0,
      messageBroker: new MessageBroker(
        this.$store.state.activeAccount,
        (data) => {
          this.$store.commit('updateMessages', data);
        },
      ),
    };
  },
  methods: {
    /** @method
     * Setter
     * Send a message to the message broker for storage and
     * Peer communication. Also send straight to the Peer2Peer handler
     * @name sendMessage
     * @argument data the data to send
     * @argument type the type of message we're broadcasting
     */
    async sendMessage(data, type) {
      if (window.Vault74.messageBroker) {
        const msg = window.Vault74.messageBroker.sentMessage(
          this.$store.state.activeChat,
          Date.now(),
          'message',
          {
            type: type || 'text',
            data,
          },
        );

        const id = this.$database.threadManager
          .makeIdentifier(this.$store.state.activeAccount, this.$store.state.activeChat);
        const threadExists = await this.$database.threadManager.fetchThread(id);
        if (threadExists) {
          const threadID = await this.$database.threadManager.threadAt(id);
          const message = {
            _id: msg.id,
            sender: msg.sender,
            at: msg.at,
            type: msg.type,
            payload: msg.payload,
          };
          // If we have their public key, we will encrypt their message
          this.$database.messageManager
            .addMessageDeterministically(threadID, message, this.$store.state.activeChat);
        }
      }
      window.Vault74.Peer2Peer.send(
        this.$store.state.activeChat,
        'message',
        {
          type: type || 'text',
          data,
        },
      );
    },
    /** @method
     * Setter
     * Reach out to the CoinCap.io API for current market
     * prices of Ethereum to USD
     * @name getMarketPrice
     */
    async getMarketPrice() {
      // pull prices from https://api.coincap.io/v2/assets/ethereum
      const response = await fetch('https://api.coincap.io/v2/assets/ethereum');
      const json = await response.json();
      this.priceUsd = json.data.priceUsd;
    },
    /** @method
     * Setter
     * Issue the transactoin on chain if everything checks out.
     * @name sendTransaction
     */
    async sendTransaction() {
      if (this.amount <= 0) {
        this.error = 'Please enter > 0 ETH.';
        return false;
      }
      this.error = false;
      ethereum.sendEther(
        this.address,
        this.$store.state.activeAccount,
        this.amount,
        (hash) => {
          this.sendMessage(
            {
              amount: this.amount,
              to: this.address,
              from: this.$store.state.activeAccount,
              tx: hash,
            },
            'payment',
          );
          this.amount = null;
          this.toggle();
        },
      );
      return true;
    },
  },
  async mounted() {
    const dwellerCachingHelper = new DwellerCachingHelper(config.registry[config.network.chain]);
    const dweller = await dwellerCachingHelper.getDweller(this.address);
    this.name = dweller.name;
    this.icon = dweller.photo;
    this.getMarketPrice();
  },
  /* eslint-disable */
  directives: {
    'click-outside': {
      bind: (el, binding) => {
        let clickedOffOnce = false;
        // Define ourClickEventHandler
        const ourClickEventHandler = event => {
          if (!el.contains(event.target) && el !== event.target) {
            if (clickedOffOnce) {
              // as we are attaching an click event listern to the document (below)
              // ensure the events target is outside the element or a child of it
              binding.value(event); // before binding it
            }
          }
          clickedOffOnce = true;
        };
        // attached the handler to the element so we can remove it later easily
        el.__vueClickEventHandler__ = ourClickEventHandler;

        // attaching ourClickEventHandler to a listener on the document here
        document.addEventListener("click", ourClickEventHandler);
      },
      unbind: function(el) {
        // Remove Event Listener
        document.removeEventListener("click", el.__vueClickEventHandler__);
      }
    }
  },
  /* eslint-enable */
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less" src="./MiniPayment.less"></style>
