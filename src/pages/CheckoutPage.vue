<template>
  <div :class="isCard ? '' : 'lg:h-screen'" class="container mx-auto p-6 grid grid-cols-1 row-gap-12 lg:grid-cols-10 lg:col-gap-10 lg:pt-12">
    <Payment @handle-card="handleCard" @change-parent="handleAlert" :total="total"/>
    <Summary :items="items"/>
    <Alert :visible="alertVisible" position="bottom-right" color="success" title="Success" description="Your payment has been successfully processed." />
  </div>
</template>

<script>
import Payment from "../components/Payment";
import Summary from "../components/Summary";
import Alert from "../components/Alert";

export default {
  name: "CheckoutPage",
  components: {
    Payment,
    Summary,
    Alert
  },
  data() {
    return {
      items: [
        {
          title: "Product 1",
          description: "lorem impsu liwe",
          price: 550,
          img: "https://picsum.photos/id/1005/600/200"
        },
        {
          title: "Product 2",
          description: "lorem impsu liwe",
          price: 250,
          img: "https://picsum.photos/id/1005/600/200"
        },
        {
          title: "Product 3",
          description: "lorem impsu liwe",
          price: 150,
          img: "https://picsum.photos/id/1005/600/200"
        }
      ],
      alertVisible: false,
      total: 0,
      isCard: false
    };
  },
  mounted() {
    this.getTotal(this.items);
  },
  methods: {
    getTotal(items) {
      items.forEach(item => {
        this.total += item.price;
      });
    },
    handleAlert() {
      this.alertVisible = true;
      setTimeout(() => {
        this.alertVisible = false;
      }, 4000);
    },
    handleCard() {
      this.isCard = true;
    }
  }
};
</script>
