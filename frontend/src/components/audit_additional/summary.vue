<template>
	<div>
		<q-card-section>
			<div class="text-h6">漏洞數量</div>
		</q-card-section>
		<q-card-section>
			<q-table :data="vuln_count" :columns="vuln_count_column"> </q-table>
		</q-card-section>
		<q-card-section>
			<div id="chart" style="width: 100%; height: 400px"></div>
		</q-card-section>
	</div>
</template>

<script>
import AuditService from "@/services/audit";

import * as echarts from "echarts";

export default {
	data: () => {
		return {
			// Set audit ID
			auditId: null,
			sectionId: null,
			vuln_count_column: [
				{
					name: "severity",
					required: true,
					label: "危險等級",
					align: "center",
					field: (row) => row.name,
				},
				{
					name: "count",
					required: true,
					label: "數量",
					align: "center",
					field: (row) => row.count,
				},
			],
			vuln_count: [],
			findings: null,
			chart_option: {
				title: {
					text: "漏洞數量",
				},
				tooltip: {
					trigger: "axis",
					axisPointer: {
						type: "shadow",
					},
				},
				grid: {
					left: "3%",
					right: "4%",
					bottom: "3%",
					containLabel: true,
				},
				xAxis: [
					{
						type: "category",
						data: ["嚴重", "高", "中", "低"],
						axisTick: {
							alignWithLabel: true,
						},
					},
				],
				yAxis: [
					{
						type: "value",
					},
				],
				series: [
					{
						name: "漏洞",
						type: "bar",
						barWidth: "60%",
						data: [0, 0, 0, 0],
					},
				],
			},
			chart_element: null,
		};
	},

	mounted: function () {
		this.auditId = this.$route.params.auditId;
		this.sectionId = this.$route.params.sectionId;
		this.getFindingsVulnCount();

		// save on ctrl+s
		document.addEventListener("keydown", this._listener, false);
	},

	destroyed: function () {
		document.removeEventListener("keydown", this._listener, false);
	},

	methods: {
		_listener: function (e) {
			if (
				(window.navigator.platform.match("Mac")
					? e.metaKey
					: e.ctrlKey) &&
				e.keyCode == 83
			) {
				e.preventDefault();
				if (this.frontEndAuditState === this.AUDIT_VIEW_STATE.EDIT)
					this.updateSection();
			}
		},

		// Get Section
		getFindingsVulnCount: function () {
			AuditService.findingsVulnCount(this.auditId)
				.then((data) => {
					let count_res = data.data.datas;
					let count_data = [
						{ name: "嚴重", count: count_res["critical"] },
						{ name: "高", count: count_res["high"] },
						{ name: "中", count: count_res["medium"] },
						{ name: "低", count: count_res["low"] },
					];
					this.vuln_count = count_data;
					this.chart_option.series[0].data = [
						{
							value: count_res["critical"],
							itemStyle: { color: "#C00000" },
						},
						{
							value: count_res["high"],
							itemStyle: { color: "#FF0000" },
						},
						{
							value: count_res["medium"],
							itemStyle: { color: "#FFBF00" },
						},
						{
							value: count_res["low"],
							itemStyle: { color: "#00b050" },
						},
					];
					this.initChart();
				})
				.catch((err) => {
					console.log(err);
				});
		},

		initChart: function (e) {
			this.chart_element = echarts.init(document.getElementById("chart"));
			this.chart_element.setOption(this.chart_option);
			this.chart_element.on("finished", () => {
				let chart_data = this.chart_element.getDataURL();
				AuditService.updateVulnChart(
					this.auditId,
					`<img class="custom-image" src="${chart_data}" alt="vulnchart"`
				);
			});
		},
	},
};
</script>
