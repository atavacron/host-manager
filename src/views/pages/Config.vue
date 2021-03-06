<template>
	<div class="page page-config">
		<div class="controls controls-header">
			<div class="control control-inline">
				<input type="checkbox" v-model="acceptContracts" id="chk-accepting-contracts" />
				<label for="chk-accepting-contracts">Accept Contracts</label>
			</div>
			<button class="btn btn-inline" @click="modal='announce'">Announce</button>
		</div>
		<div class="host-config-header">
			<div>Current</div>
			<div>Average</div>
		</div>
		<div class="host-config">
			<div class="config-header">Financials</div>
			<config-item title="Contract Price"
				configKey="mincontractprice"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['mincontractprice'] != null"
				:value="currentConfig.contractPrice"
				:avgValue="formatPriceString(averageSettings.contract_price, 2)"
				:error="errors['mincontractprice']"
				description="The amount of money to form a contract
					with the host."
				@change="onChangePrice" />
			<config-item title="Storage Price"
				configKey="minstorageprice"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['minstorageprice'] != null"
				:value="currentConfig.storagePrice"
				:avgValue="formatMonthlyPriceString(averageSettings.storage_price, 2)"
				:sublabel="perMonthLabel"
				:error="errors['minstorageprice']"
				:description="storageDesc"
				@change="onChangeMonthlyPrice" />
			<config-item title="Download Price"
				configKey="mindownloadbandwidthprice"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['mindownloadbandwidthprice'] != null"
				:value="currentConfig.downloadPrice"
				:avgValue="formatDataPriceString(averageSettings.download_price, 2)"
				:sublabel="dataLabel"
				:error="errors['mindownloadbandwidthprice']"
				:description="downloadDesc"
				@change="onChangeDataPrice" />
			<config-item title="Upload Price"
				configKey="minuploadbandwidthprice"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['minuploadbandwidthprice'] != null"
				:value="currentConfig.uploadPrice"
				:avgValue="formatDataPriceString(averageSettings.upload_price, 2)"
				:sublabel="dataLabel"
				:error="errors['minuploadbandwidthprice']"
				:description="uploadDesc"
				@change="onChangeDataPrice" />
			<config-item title="Base RPC Price"
				configKey="minbaserpcprice"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['minbaserpcprice'] != null"
				:value="currentConfig.baseRPCPrice"
				:avgValue="formatPriceString(averageSettings.base_rpc_price, 2)"
				sublabel="per request"
				:error="errors['minuploadbandwidthprice']"
				description="The amount of money that the renter will be
					charged to for each upload, download, and contract formation request to the host."
				@change="onChangePrice" />
			<config-item title="Sector Access Price"
				configKey="minsectoraccessprice"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['minsectoraccessprice'] != null"
				:value="currentConfig.sectorAccessPrice"
				:avgValue="formatPriceString(averageSettings.sector_access_price, 2)"
				sublabel="per sector"
				:error="errors['minuploadbandwidthprice']"
				description="The amount of money that the renter will be charged to read a
					single sector of data from disk. The host has to read at least one full 4MB sector
					from disk regardless of how much the renter intends to download
					this is charged to pay for the physical disk resources the renter consumes."
				@change="onChangePrice" />
			<div class="config-header">Collateral</div>
			<config-item title="Collateral Budget"
				configKey="collateralbudget"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['collateralbudget'] != null"
				:value="currentConfig.collateralBudget"
				:error="errors['collateralbudget']"
				description="The maximum amount of collateral
					the host will lock for all contracts. When this is full the host will reject new
					contracts"
				@change="onChangePrice" />
			<config-item title="Max Collateral"
				configKey="maxcollateral"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['maxcollateral'] != null"
				:value="currentConfig.maxCollateral"
				:avgValue="formatPriceString(averageSettings.max_collateral, 2)"
				sublabel="per contract"
				:error="errors['maxcollateral']"
				description="The maximum amount of collateral
					that the host will risk per contract. If an individual contract reaches this maximum
					the host will refuse to accept any more data from that contract."
				@change="onChangePrice" />
			<config-item title="Collateral"
				configKey="collateral"
				:showPin="true"
				:pinned="appConfig.host_pricing_pins && appConfig.host_pricing_pins['collateral'] != null"
				:value="currentConfig.collateral"
				:avgValue="formatMonthlyPriceString(averageSettings.collateral, 2)"
				:sublabel="perMonthLabel"
				:error="errors['collateral']"
				:description="collateralDesc"
				@change="onChangeMonthlyPrice" />
			<div class="config-header">Duration</div>
			<config-item title="Max Duration"
				configKey="maxduration"
				:value="currentConfig.maxDuration"
				:avgValue="formatBlockTimeString(averageSettings.max_duration)"
				:error="errors['maxduration']"
				description="The maximum amount of time the host will
					accept for a storage contract. Hosts are only payed at the end of a storage contract."
				@change="onChangeTime" />
			<config-item title="Proof Window Duration"
				configKey="windowsize"
				:value="currentConfig.windowSize"
				:avgValue="formatBlockTimeString(averageSettings.window_size)"
				:error="errors['windowsize']"
				description="The amount of time the host has to submit
					a storage proof when the contract expires."
				@change="onChangeTime" />
			<div class="config-header">Network</div>
			<config-item title="Download Batch Size"
				configKey="maxdownloadbatchsize"
				:value="currentConfig.maxDownloadSize"
				:avgValue="formatByteString(averageSettings.max_download_batch_size, 2)"
				:error="errors['maxdownloadbatchsize']"
				description="The maximum size of a single download request from
					the renter. Larger batch sizes mean fewer round trips and better performance,
					but more financial risk for the host."
				@change="onChangeBytes" />
			<config-item title="Revise Batch Size"
				configKey="maxrevisebatchsize"
				:value="currentConfig.maxReviseSize"
				:avgValue="formatByteString(averageSettings.max_revise_batch_size, 2)"
				:error="errors['maxrevisebatchsize']"
				description="The maximum size of a single file contract revision.
					Larger batch sizes allow for higher throughput of renters, but more financial risk for the host."
				@change="onChangeBytes" />
		</div>
		<div class="controls">
			<button class="btn btn-success btn-inline" @click="onClickUpdate" :disabled="!changed || updating">Update Configuration</button>
		</div>
		<announce-host-modal v-if="modal === 'announce'" @close="onModalClose" />
	</div>
</template>

<script>
import log from 'electron-log';

import { mapState, mapActions } from 'vuex';
import { writeConfig } from '@/utils';
import { formatByteString, formatBlockTimeString, formatPriceString, formatDataPriceString, formatMonthlyPriceString } from '@/utils/format';
import { parseCurrencyString, parseByteString, parseBlockTimeString } from '@/utils/parse';
import SiaApiClient from '@/sia/api';
import { refreshHostConfig } from '@/sync/config';

import AnnounceHostModal from '@/components/config/AnnounceHostModal';
import ConfigItem from '@/components/config/ConfigItem';

export default {
	components: {
		AnnounceHostModal,
		ConfigItem
	},
	computed: {
		...mapState({
			appConfig: state => state.config,
			hostConfig: state => state.hostConfig.config,
			averageSettings: state => state.explorer.averageSettings
		}),
		storageDesc() {
			const dataUnit = this.appConfig.data_unit === 'decimal' ? 'TB' : 'TiB';

			return `The amount of money that the renter will be charged for storing 1 ${dataUnit} of
				data for a month. Payment is not received until after the contract has expired and a
				valid proof has been submitted.`;
		},
		uploadDesc() {
			const dataUnit = this.appConfig.data_unit === 'decimal' ? 'TB' : 'TiB';

			return `The amount of money that the renter will be charged to upload 1 ${dataUnit} of data
				to store. Payment is not received until after the contract has expired and a valid
				proof has been submitted.`;
		},
		downloadDesc() {
			const dataUnit = this.appConfig.data_unit === 'decimal' ? 'TB' : 'TiB';

			return `The amount of money that the renter will be
				charged to download 1 ${dataUnit} of stored data. Payment is not received until after
				the contract has expired and a valid proof has been submitted.`;
		},
		collateralDesc() {
			const dataUnit = this.appConfig.data_unit === 'decimal' ? 'TB' : 'TiB';

			return `The amount of collateral the host will
				risk for 1 ${dataUnit} of stored data per month. A good amount is 2x Storage Price.
				Any risked collateral will be lost if a valid storage proof is not
				submitted after contract expiration`;
		},
		perMonthLabel() {
			const dataUnit = this.appConfig.data_unit === 'decimal' ? 'TB' : 'TiB';

			return `per ${dataUnit}/month`;
		},
		dataLabel() {
			const dataUnit = this.appConfig.data_unit === 'decimal' ? 'TB' : 'TiB';

			return `per ${dataUnit}`;
		}
	},
	data() {
		return {
			acceptContracts: false,
			errors: {},
			config: {},
			currentConfig: {},
			pinned: {},
			updating: false,
			changed: false,
			modal: null
		};
	},
	mounted() {
		this.updateConfig();
	},
	methods: {
		...mapActions(['pushNotification', 'setConfig']),
		formatBlockTimeString,
		formatByteString,
		formatPriceString,
		formatDataPriceString,
		formatMonthlyPriceString,
		async onModalClose() {
			try {
				this.modal = null;
				this.updating = true;

				await refreshHostConfig();
				this.updateConfig();
			} catch (ex) {
				log.error('config modal close', ex.message);
			} finally {
				this.updating = false;
			}
		},
		updateConfig() {
			const byteFactor = this.appConfig.data_unit === 'decimal' ? 1e12 : 1099511627776;

			this.acceptContracts = this.hostConfig.acceptingcontracts;

			this.currentConfig = {
				contractPrice: formatPriceString(this.hostConfig.mincontractprice, 2),
				storagePrice: formatPriceString(this.hostConfig.minstorageprice.times(byteFactor).times(4320), 2),
				downloadPrice: formatPriceString(this.hostConfig.mindownloadbandwidthprice.times(byteFactor), 2),
				uploadPrice: formatPriceString(this.hostConfig.minuploadbandwidthprice.times(byteFactor), 2),
				collateralBudget: formatPriceString(this.hostConfig.collateralbudget, 2),
				baseRPCPrice: formatPriceString(this.hostConfig.minbaserpcprice, 2),
				sectorAccessPrice: formatPriceString(this.hostConfig.minsectoraccessprice, 2),
				maxCollateral: formatPriceString(this.hostConfig.maxcollateral, 2),
				collateral: formatPriceString(this.hostConfig.collateral.times(byteFactor).times(4320), 2),
				maxDuration: formatBlockTimeString(this.hostConfig.maxduration),
				windowSize: formatBlockTimeString(this.hostConfig.windowsize),
				maxDownloadSize: formatByteString(this.hostConfig.maxdownloadbatchsize, 2),
				maxReviseSize: formatByteString(this.hostConfig.maxrevisebatchsize, 2)
			};
		},
		async onClickUpdate() {
			if (this.updating)
				return;

			try {
				this.updating = true;

				const client = new SiaApiClient(this.appConfig);

				await client.updateHost(this.config);

				const configPins = { ...this.appConfig.host_pricing_pins };

				for (let pin in this.pinned) {
					if (!pin)
						continue;

					if (this.pinned[pin] === false) {
						configPins[pin] = null;
						continue;
					}

					configPins[pin] = this.pinned[pin];
				}

				this.setConfig({ host_pricing_pins: configPins });

				await refreshHostConfig();
				this.updateConfig();
				await writeConfig(this.appConfig);

				this.pushNotification({
					message: 'Host Configuration Updated',
					icon: 'cogs',
					style: 'success'
				});
			} catch (ex) {
				this.pushNotification({
					message: ex.message,
					icon: 'cogs',
					style: 'danger'
				});
			} finally {
				this.updating = false;
				this.changed = false;
			}
		},
		onChangePrice(obj) {
			const { key, value, pinned } = obj;

			try {
				const val = parseCurrencyString(value);
				let pin = null;

				this.config[key] = val.toFixed(0).toString(10);

				if (pinned) {
					pin = {
						currency: this.appConfig.currency,
						value: value
					};
				}

				this.changed = true;
				this.$set(this.pinned, key, pin);
				this.$set(this.errors, key, null);
			} catch (ex) {
				this.$set(this.errors, key, ex.message);
			}
		},
		onChangeMonthlyPrice(obj) {
			const { key, value, pinned } = obj;

			try {
				const val = parseCurrencyString(value),
					byteFactor = this.appConfig.data_unit === 'decimal' ? 1e12 : 1099511627776;
				let pin = null;

				this.config[key] = val.div(byteFactor).div(4320).toFixed(0).toString(10);

				if (pinned) {
					pin = {
						currency: this.appConfig.currency,
						value: value
					};
				}

				this.changed = true;
				this.$set(this.pinned, key, pin);
				this.$set(this.errors, key, null);
			} catch (ex) {
				this.$set(this.errors, key, ex.message);
			}
		},
		onChangeDataPrice(obj) {
			const { key, value, pinned } = obj;

			try {
				const val = parseCurrencyString(value),
					byteFactor = this.appConfig.data_unit === 'decimal' ? 1e12 : 1099511627776;
				let pin = null;

				this.config[key] = val.div(byteFactor).toFixed(0).toString(10);

				if (pinned) {
					pin = {
						currency: this.appConfig.currency,
						value: value
					};
				}

				this.changed = true;
				this.$set(this.pinned, key, pin);
				this.$set(this.errors, key, null);
			} catch (ex) {
				this.$set(this.errors, key, ex.message);
			}
		},
		onChangeTime(obj) {
			const { key, value } = obj;

			try {
				this.config[key] = parseBlockTimeString(value);
				this.$set(this.errors, key, null);

				this.changed = true;
			} catch (ex) {
				this.$set(this.errors, key, ex.message);
			}
		},
		onChangeBytes(obj) {
			const { key, value } = obj;

			try {
				this.config[key] = parseByteString(value).toString(10);
				this.$set(this.errors, key, null);

				this.changed = true;
			} catch (ex) {
				this.$set(this.errors, key, ex.message);
			}
		}
	},
	watch: {
		acceptContracts(value, oldvalue) {
			if (value === oldvalue)
				return;

			this.config['acceptingcontracts'] = value;
			this.changed = true;
		}
	}
};
</script>

<style lang="stylus" scoped>
.page-config {
	display: grid;
	grid-template-rows: repeat(2, auto) minmax(0, 1fr) auto;
}

.host-config-header {
    display: grid;
    grid-template-columns: 2fr 1fr;
    grid-gap: 30px;
    padding: 15px 30px 5px;
    text-align: right;
    color: rgba(255, 255, 255,0.54);
}

.config-header {
    margin-bottom: 15px;
    font-size: 1rem;
    color: rgba(255, 255, 255, 0.54);
    margin-top: 45px;

    &:first-of-type {
        margin-top: 0;
    }
}

.controls {
	padding: 15px 5px;
	text-align: center;

	&.controls-header {
		text-align: right;
	}

	.control {
		margin-bottom: 0;
		margin-right: 15px;
	}
}

.host-config {
	width: 100%;
	height: 100%;
	padding: 15px;
	overflow-y: auto;
}
</style>
