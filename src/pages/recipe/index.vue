<template>
  <view class="recipe-list">
    <view class="recipe-grid">
      <view 
        class="recipe-item" 
        v-for="recipe in recipes" 
        :key="recipe.id"
        @tap="goToDetail(recipe.id)"
      >
        <image 
          class="recipe-cover" 
          :src="recipe.coverImage" 
          mode="aspectFill"
        />
        <view class="recipe-info">
          <text class="recipe-name">{{ recipe.name }}</text>
          <text class="recipe-time">预计耗时: {{ recipe.cookingTime }}分钟</text>
        </view>
      </view>
    </view>

    <!-- 加载更多 -->
    <uni-load-more 
      :status="loadMoreStatus" 
      :content-text="loadMoreText"
    />

    <!-- 悬浮添加按钮 -->
    <view 
      class="add-btn"
      @tap="goToAdd"
    >
      <text class="iconfont icon-add"></text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      page: 1,
      recipes: [],
      hasMore: true,
      loadMoreStatus: 'more',
      loadMoreText: {
        contentdown: '上拉加载更多',
        contentrefresh: '加载中...',
        contentnomore: '没有更多数据了'
      }
    }
  },
  
  onLoad() {
    this.fetchRecipes()
  },
  
  // 下拉刷新
  onPullDownRefresh() {
    this.page = 1
    this.recipes = []
    this.hasMore = true
    this.fetchRecipes().then(() => {
      uni.stopPullDownRefresh()
    })
  },
  
  // 上拉加载更多
  onReachBottom() {
    if (this.hasMore) {
      this.page++
      this.fetchRecipes()
    }
  },
  
  methods: {
    async fetchRecipes() {
      if (!this.hasMore) return
      
      this.loadMoreStatus = 'loading'
      try {
        // 这里替换为实际的API调用
        const res = await uni.request({
          url: `/api/recipes`,
          data: {
            page: this.page,
            pageSize: 10
          }
        })
        
        const { items, hasMore } = res.data
        
        this.recipes = this.page === 1 
          ? items 
          : [...this.recipes, ...items]
          
        this.hasMore = hasMore
        this.loadMoreStatus = this.hasMore ? 'more' : 'noMore'
      } catch (error) {
        console.error('获取菜谱列表失败:', error)
        this.loadMoreStatus = 'more'
      }
    },
    
    goToDetail(id) {
      uni.navigateTo({
        url: `/pages/recipe/detail?id=${id}`
      })
    },
    
    goToAdd() {
      uni.navigateTo({
        url: '/pages/recipe/add'
      })
    }
  }
}
</script>

<style lang="scss">
.recipe-list {
  padding: 30rpx;
  
  .recipe-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 30rpx;
  }
  
  .recipe-item {
    background: #fff;
    border-radius: 12rpx;
    overflow: hidden;
    box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.1);
    
    .recipe-cover {
      width: 100%;
      height: 200rpx;
    }
    
    .recipe-info {
      padding: 20rpx;
      
      .recipe-name {
        font-size: 28rpx;
        font-weight: bold;
        color: #333;
      }
      
      .recipe-time {
        font-size: 24rpx;
        color: #999;
        margin-top: 10rpx;
      }
    }
  }
  
  .add-btn {
    position: fixed;
    right: 40rpx;
    bottom: 40rpx;
    width: 100rpx;
    height: 100rpx;
    background: #007AFF;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 4rpx 16rpx rgba(0, 122, 255, 0.3);
    
    .iconfont {
      color: #fff;
      font-size: 50rpx;
    }
  }
}
</style> 