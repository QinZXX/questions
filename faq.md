求一个最大连续子串和

int getMaxSum(const vector<int>& nums) {
    int cur = 0; // 以 i 结尾的连续子串最大和
    int res = 0; // 已知的最大和
    for (size_t i = 0; i < nums.size(); ++i) {
        cur = max<int>(cur + nums[i], 0); // 子串和最小为 0
        res = max<int>(cur, res); // 记录已知的最大和
    }

    return res;
}

// 测试
// 1 2 -4 5
// cur 1 3 0(-1) 5 => res = 5

// 1 0 1 0 -3 4 =>
// cur 1 0 2 0 0(-1) 4 => res = 4;

