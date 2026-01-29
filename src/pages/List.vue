<template>
    <transition name="slide-fade" appear>
        <MonitorList :scrollbar="true" />
    </transition>
</template>

<script>
import MonitorList from "../components/MonitorList.vue";

export default {
    components: {
        MonitorList,
    },
    watch: {
        "$root.isMobile"(newVal) {
            if (!newVal && this.$route.path === "/kuma-list") {
                this.$router.replace({ path: "/kuma-dashboard" });
            }
        },
    },
    mounted() {
        if (!this.$root.isMobile && this.$route.path === "/kuma-list") {
            this.$router.replace({ path: "/kuma-dashboard" });
        }
    },
};
</script>

<style lang="scss" scoped>
@import "../assets/vars";

.shadow-box {
    padding: 20px;
}
</style>
