<template>
	<view>
		<view v-if="onload" style="text-align: center;padding-top: 30vh;">
			<image class="loading" src="@/static/icon/loading.png"></image>
			<br>
			<text>{{ $t('loading1') }}</text>
		</view>
		<view v-else>
			<view v-if="error" style="text-align: center;padding-top: 30vh;">
				<image src="@/static/icon/cactus.png" style="width: 4rem;height: 4rem;" />
				<view style="font-size: 1.3rem;font-weight: bold;;color: #00897B;margin: 0.5rem 0;">
					<text>{{ $t('search.notData.excerpt') }}</text>
				</view>
				<text>{{ $t('search.notData.image') }}</text>
			</view>
			<view v-else>
				<lists :data="data" ref="lists" type="video"></lists>
				<view style="text-align: center;padding-bottom: 1rem;" dir="auto">
					<text v-if="loading">
						<i class="fa-solid fa-circle-notch fa-spin" style="color: #00897b;margin-right: 0.4rem;"></i>
						<text>{{ $t('loading2') }}</text>
					</text>
					<text v-else>
						<text><i style="transform:scale(2.5,1)" class="fa-solid fa-minus"></i></text>
					</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import lists from "@/pages/lists/index.vue";
import { search, fill0 } from "@/api/api.js";
export default {
	components: {
		lists,
	},
	data() {
		return {
			data: [],
			page: 0,
			onload: true,
			loading: false,
			error: false
		}
	},
	props: ['s'],
	created() {
		this.getData(() => {
			this.onload = false
		})
	},
	methods: {
		// 刷新
		refresh(cb) {
			this.onload = true
			this.data = []
			this.page = 0
			this.getData(() => {
				this.onload = false
				cb()
			})
		},
		onBottom() {
			console.log(this.error)
			console.log(this.loading)
			if (this.error == false && this.loading == false) {
				this.getData(() => { })
			}
		},
		// 获取列表
		getData(cb) {
			this.loading = true
			search('video', this.s, this.page, (res, code) => {
				this.loading = false
				if (code == 200) {
					if (res.results.length > 0) {
						this.setPageData(res, code);
						this.page++
					} else {
						if (this.page == 0) {
							this.error = true
						}
					}
				} else if (code == 408) {
					uni.showToast({
						title: this.$t('dontLink'),
						icon: "none",
						duration: 3000,
					});
				} else { }
				cb()
			})
		},
		// 生成列表
		setPageData(res) {
			for (let i = 0; i < res.results.length; i++) {
				let rs = res.results[i];
				this.data.push({
					id: rs.id,
					label: rs.title,
					img:
						rs.file != null
							? "https://iwaraapi.pro-ivan.cn/imgs/image/thumbnail/" +
							rs.file.id +
							"/thumbnail-" + fill0(rs.thumbnail, 1) + ".jpg"
							: null,
					date: this.formatDate(rs.createdAt),
					author: rs.user.name,
					avatar:
						rs.user.avatar != null
							? "https://iwaraapi.pro-ivan.cn/imgs/image/avatar/" +
							rs.user.avatar.id +
							"/" +
							rs.user.avatar.name
							: "https://www.iwara.tv/images/default-avatar.jpg",
					watch: rs.numViews,
					like: rs.numLikes,
					uid: rs.user.id,
					username: rs.user.username
				});
			}
		},
		// 时间格式化
		formatDate(t) {
			let d = new Date();
			let year = t.slice(0, 4);
			let month = t.slice(5, 7);
			let day = t.slice(8, 10);
			if (d.getFullYear() == year) {
				t = month + "-" + day;
			} else {
				t = year + "-" + month + "-" + day;
			}
			return t;
		},
	}
}
</script>

<style scoped>
.loading {
	width: 4rem;
	height: 4rem;
	animation: rotate 350ms linear infinite;
}

@keyframes rotate {
	0% {
		transform: rotate(0deg);
	}

	25% {
		transform: rotate(90deg);
	}

	50% {
		transform: rotate(180deg);
	}

	75% {
		transform: rotate(270deg);
	}

	100% {
		transform: rotate(360deg);
	}
}
</style>
