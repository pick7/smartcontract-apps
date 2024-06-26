# 不受单个矿工控制的链上随机数生成方法

## 备注

时间：2024 年 4 月 23 日

作者：[33357](https://github.com/33357)

## 正文

如何在区块链上生成随机数一直是一个问题，一个常用的方法是通过区块 hash 生成，但是矿工可以通过改变打包内容来改变区块 hash，从而控制随机数。

这个问题其中一个解决办法是使用 chainlink 提供的链外随机数，但需要额外花钱，有没有更好的解决办法呢？

如果对以上方案进行妥协，让单个矿工对随机数生成的控制降到最小，那么也可以认为随机数在一定范围内是可靠的。

### 设计

既然单个矿工可以控制区块 hash，那么可以使用多个区块 hash 来生成随机数。uint256 有 256 位，可以使用最近的 256 个块的 hash 来组成随机数，并且单个区块 hash 只能决定 256 bit 中的 1 bit。

### 源码

``` javascript
pragma solidity ^0.8.20;

contract Random {
    // 获取随机数，只能用在 256 个块之前的行为
    function getRandomUsedBefore256() external view returns (uint256 random) {
        // 遍历最近的 256 个块
        for (uint256 i; i < 256; i++) {
            // 获得 (block.number - i) 区块的 hash 的最后 1 位数
            uint256 bit = uint256(blockhash(block.number - i)) & 1;
            // 最后 1 位数依次赋值给 random
            random |= (bit << i);
        }
    }
}
```

### 总结

在以上实现中，单个区块的 hash 值只能影响 1 个 bit 的数据。矿工只在 0 和 1 两个结果中选择，对链上随机数生成的影响降到了最小。但由于需要未来 256 个块的 hash 才能生成随机数，因此在所有的投注行为完成之后，必须等待 256 个块才能取随机数，不然随机性是不足的。

需要注意的是，以上所有理论建立在区块生成者是复数并且随机的基础上，如果是单机链，就没有讨论的必要了。
