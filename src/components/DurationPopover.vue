<template>
  <!-- TODO Make values dynamic and internationalise text  -->
  <ion-select :interface-options="customPopoverOptions" interface="popover" :value="getJobStatus(id)" @ionChange="updateJob($event['detail'].value, id)" >
    <ion-select-option value="EVERY_5_MIN">Every 5 minutes</ion-select-option>
    <ion-select-option value="EVERY_15_MIN">Every 15 minutes</ion-select-option>
    <ion-select-option value="HOURLY">Hourly</ion-select-option>
    <ion-select-option value="DAILY">Daily</ion-select-option>
    <ion-select-option value="SERVICE_DRAFT">Disabled</ion-select-option>
  </ion-select>
</template>
<script lang="ts">
import { IonSelect, IonSelectOption } from '@ionic/vue';
import { defineComponent } from 'vue';
import { useStore } from "@/store";
import { mapGetters } from "vuex";
import { translate } from '@/i18n';

export default defineComponent({
  name: 'DurationPopover',
  props: { 
    id: String,
    frequencyType: String
  },
  components: {
    IonSelect,
    IonSelectOption
  },
  computed: {
    ...mapGetters({
      getJobStatus: 'job/getJobStatus',
      getJob: 'job/getJob',
      getShopifyConfigId: 'user/getShopifyConfigId',
      getCurrentEComStore: 'user/getCurrentEComStore'
    })
  },
  methods: {
    async updateJob(status: string, id: string) {
      const job = this.getJob(id);

      // TODO: added this condition to not call the api when the value of the select automatically changes
      // need to handle this properly
      if (status === job?.tempExprId) {
        return;
      }

      const payload = {
        ...job,
        'systemJobEnumId': id,
        'statusId': status === "SERVICE_DRAFT" ? "SERVICE_CANCELLED" : "SERVICE_PENDING"
      } as any
      if (status === 'SERVICE_DRAFT') {
        this.store.dispatch('job/updateJob', payload)
      } else if (job?.status === 'SERVICE_DRAFT') {
        payload['SERVICE_FREQUENCY'] = status
        payload['SERVICE_NAME'] = job.serviceName
        payload['count'] = -1
        payload['runAsSystem'] = true
        payload['shopifyConfigId'] = this.getShopifyConfigId
        payload['productStoreId'] = this.getCurrentEComStore.productStoreId

        this.store.dispatch('job/scheduleService', payload)
      } else if (job?.status === 'SERVICE_PENDING') {
        payload['tempExprId'] = status === 'SERVICE_DRAFT' ? job.tempExprId : status
        payload['jobId'] = job.id

        this.store.dispatch('job/updateJob', payload)
      }
    }
  },
 setup(){
    const customPopoverOptions: any = {
      header: translate('Import new orders'),
      showBackdrop: false
    };
    const store = useStore();
    return {
      customPopoverOptions,
      store
    }
 }
});
</script>