<template>
  <v-container v-show="$store.getters.isUserLoggedIn">
    <delete-dialog
      :id-to-delete="selectedItem.id"
      :show-delete-dialog="showDeleteDialog"
      @on-cancel-btn-pressed="handleCancelBtnPressed()"
      @on-confirm-delete-btn-pressed="
        handleConfirmDeleteBtnPressed(selectedItem)
      "
    />
    <v-card-title>
      {{ $t("pending.title") }}
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Search pending submissions"
        single-line
        filled
        outlined
        clearable
      ></v-text-field>
    </v-card-title>
    <v-data-table
      :headers="headerList"
      :items="clinics"
      :items-per-page="10"
      :search="search"
      class="elevation-1"
    >
      <template v-slot:[`item.action`]="{ item }">
        <v-dialog v-model="dialogEdit" max-width="500px">
          <v-card>
            <v-card-title>{{ $t("pending.edit") }}</v-card-title>
            <v-col cols="12">
              <v-text-field
                v-for="(value, name) in selectedItem"
                :key="name"
                :label="name"
                :value="value"
                v-model="editedItem[name]"
              ></v-text-field>
            </v-col>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="primary"
                text
                @click="handleApproveBtnPressed(item)"
              >
                {{ $t("general.approve") }}
              </v-btn>
              <v-btn color="primary" text @click="handleCancelEditBtnPressed">
                {{ $t("general.cancel") }}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!--eslint-disable-next-line @intlify/vue-i18n/no-raw-text-->
        <v-icon small class="mr-2" @click="handleEditBtnPressed(item)">
          mdi-pencil
        </v-icon>
        <!--eslint-disable-next-line @intlify/vue-i18n/no-raw-text-->
        <v-icon small @click="handleDeleteBtnPressed(item)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import DeleteDialog from "./DeleteDialog.vue";
import json from "../data/adminDbTableHeaders.json";
import logger from "../services/logger";
export default {
  components: { DeleteDialog },
  data() {
    return {
      editedItem: {},
      clinics: [],
      dialogEdit: false,
      dialogDelete: false,
      headerList: json.headers,
      search: "",
      showDeleteDialog: false,
      selectedItem: {
        id: "",
      },
      pendingItem: {
        name: "",
        prefecture: "",
        city: "",
        ward: "",
        note: "",
        voucherRequired: "",
        wardResidencyRequired: "",
        website: "",
      },
    };
  },
  mounted() {
    if (!this.$store.getters.isUserLoggedIn) {
      this.$router.push("/login");
    }
    this.getPendingSubmissions();
  },
  methods: {
    async getPendingSubmissions() {
      const db = this.$fireModule.firestore();
      let pendingRef = db.collection("pending");
      let pendingClinics = await pendingRef.get();
      for (const doc of pendingClinics.docs) {
        this.id = doc.id;
        this.clinics.push(doc.data());
        this.clinics[this.clinics.length - 1].id = this.id;
      }
    },
    async approve(item) {
      try {
        await this.$fireModule
          .firestore()
          .collection("clinics")
          .add(this.editedItem);
        logger.info("submitted: ", item);
      } catch (err) {
        logger.error(err);
      }
    },
    editItem(item) {
      for (const key in item) {
        this.pendingItem[key] = item[key];
      }
      this.editedItem = Object.assign(item);
    },
    async deleteItem(item) {
      const itemIndex = this.clinics.indexOf(item);
      try {
        await this.$fireModule
          .firestore()
          .collection("pending")
          .doc(item.id)
          .delete();
        await this.clinics.splice(itemIndex, 1);
      } catch (err) {
        logger.error(err);
      }
    },
    async handleApproveBtnPressed(item) {
      this.dialogEdit = false;
      await this.approve(item);
    },
    async handleEditBtnPressed(item) {
      this.dialogEdit = true;
      this.selectedItem = item;
      await this.editItem(item);
    },
    handleCancelEditBtnPressed() {
      this.dialogEdit = false;
      for (const key in this.pendingItem) {
        this.editedItem[key] = this.pendingItem[key];
      }
    },
    handleDeleteBtnPressed(item) {
      this.selectedItem = item;
      this.showDeleteDialog = true;
    },
    async handleConfirmDeleteBtnPressed(item) {
      await this.deleteItem(item);
      this.showDeleteDialog = false;
    },
    handleCancelBtnPressed() {
      this.showEditDialog = false;
      this.showDeleteDialog = false;
    },
  },
};
</script>
