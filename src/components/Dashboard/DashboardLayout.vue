<template>
  <div class="dashboard-layout">
    <SidebarNavigation
      @connect-wallet="connectToWallet"
      @register-member="registerMember"
      @select="handleComponentSelection"
      :is-member="isMember"
    />
    <section class="dashboard-content">
      <!-- Connection and registration status messages -->
      <div v-if="isConnected" class="connection-status">
        You are connected with address: {{ connectedAddress }}
      </div>
      <div v-if="connectionError" class="connection-status error">
        Failed to connect: {{ connectionError }}
      </div>
      <div v-if="registrationSuccess" class="connection-status">
        Registration successful!
      </div>
      <component :is="currentComponent" />
    </section>
  </div>
</template>

<script>
import SidebarNavigation from "./SideBarNavigation.vue";
import ConnectWallet from "../ConnectWallet.vue";
import CreateProposal from "../proposals/ProposalCreation.vue";
import ViewList from "../proposals/ViewList.vue";
import ApproveProposals from "../approveProposal/ApproveProposal.vue";
import { connectWallet } from "../Web3/blockchainService.js";
import { registerMember, checkIfMember } from "../Web3/daoContractService";

export default {
  components: {
    SidebarNavigation,
    ConnectWallet,
    CreateProposal,
    ViewList,
    ApproveProposals,
  },
  data() {
    return {
      currentComponent: null,
      connectedAddress: "",
      errorMessage: "",
      isConnected: false,
      registrationSuccess: false,
      isMember: false,
      connectionError: "",
    };
  },
  methods: {
    async connectToWallet() {
      try {
        await connectWallet();
        this.connectedAddress = await this.getCurrentAddress();
        this.isConnected = true;
        this.connectionError = "";
        this.isMember = await checkIfMember(this.connectedAddress);
      } catch (error) {
        console.error("Wallet connection failed:", error);
        this.connectionError = error.message;
        this.isConnected = false;
      }
    },
    async getCurrentAddress() {
      if (window.ethereum) {
        const accounts = await window.ethereum.request({
          method: "eth_accounts",
        });
        return accounts[0] || "";
      }
      return "";
    },
    async registerMember() {
      try {
        if (!this.isConnected) {
          throw new Error("Please connect your wallet first.");
        }
        await registerMember();
        this.registrationSuccess = true;
        setTimeout(() => (this.registrationSuccess = false), 3000);
        this.isMember = true;
      } catch (error) {
        console.error("Registration failed:", error);
        this.errorMessage = error.message;
      }
    },
    handleComponentSelection(componentName) {
      const componentMap = {
        ApproveProposals: ApproveProposals,
        ViewList: ViewList,
      };
      this.currentComponent = componentMap[componentName];
    },
  },
};
</script>

<style scoped>
.dashboard-layout {
  display: flex;
  height: 100vh;
}

.dashboard-content {
  flex-grow: 1;
  padding: 20px;
  background-color: #1e1e1e;
  color: #fff;
  box-sizing: border-box;
}

.connection-status {
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 4px;
  color: #fff;
}

.connection-status.error {
  background-color: #ff3b30; /* Red for errors */
}

/* ... other styles ... */
</style>