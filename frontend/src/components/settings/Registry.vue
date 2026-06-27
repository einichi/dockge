<template>
    <div>
        <form class="my-4" autocomplete="off" @submit.prevent="login">
            <div class="mb-4">
                <label for="registry-endpoint" class="form-label">
                    {{ $t("dockgeAgent") }}
                </label>
                <select
                    id="registry-endpoint"
                    v-model="endpoint"
                    class="form-select"
                    :disabled="loggingIn"
                >
                    <option
                        v-for="agent in endpointList"
                        :key="agent.endpoint"
                        :value="agent.endpoint"
                    >
                        {{ endpointDisplay(agent) }}
                    </option>
                </select>
            </div>

            <div class="mb-4">
                <label for="registry-server" class="form-label">
                    {{ $t("registryServer") }}
                </label>
                <input
                    id="registry-server"
                    v-model="registryServer"
                    type="text"
                    class="form-control"
                    placeholder="registry.example.com"
                    autocapitalize="none"
                    autocomplete="off"
                    spellcheck="false"
                    :disabled="loggingIn"
                />
            </div>

            <div class="mb-4">
                <label for="registry-username" class="form-label">
                    {{ $t("Username") }}
                </label>
                <input
                    id="registry-username"
                    v-model="username"
                    type="text"
                    class="form-control"
                    autocapitalize="none"
                    autocomplete="off"
                    spellcheck="false"
                    required
                    :disabled="loggingIn"
                />
            </div>

            <div class="mb-4">
                <label for="registry-password" class="form-label">
                    {{ $t("registryPassword") }}
                </label>
                <input
                    id="registry-password"
                    v-model="password"
                    type="password"
                    class="form-control"
                    autocomplete="new-password"
                    required
                    :disabled="loggingIn"
                />
            </div>

            <div>
                <button class="btn btn-primary" type="submit" :disabled="loggingIn">
                    <font-awesome-icon v-if="loggingIn" icon="spinner" spin />
                    <font-awesome-icon v-else icon="warehouse" />
                    {{ $t("dockerRegistryLogin") }}
                </button>
            </div>
        </form>
    </div>
</template>

<script>
export default {
    name: "Registry",

    data() {
        return {
            endpoint: "",
            registryServer: "",
            username: "",
            password: "",
            loggingIn: false,
        };
    },

    computed: {
        endpointList() {
            return Object.values(this.$root.agentList);
        },
    },

    methods: {
        endpointDisplay(agent) {
            if (!agent.endpoint) {
                return this.$t("currentEndpoint");
            }
            return agent.name || agent.endpoint;
        },

        login() {
            this.loggingIn = true;
            const timeout = setTimeout(() => {
                this.loggingIn = false;
                this.$root.toastError("dockerRegistryLoginTimedOut");
            }, 30000);

            this.$root.emitAgent(this.endpoint, "dockerLogin", this.registryServer, this.username, this.password, (res) => {
                clearTimeout(timeout);
                this.loggingIn = false;
                this.$root.toastRes(res);

                if (res.ok) {
                    this.password = "";
                }
            });
        },
    },
};
</script>
