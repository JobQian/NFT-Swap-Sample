// SPDX-License-Identifier: MIT
pragma solidity ^0.8.21;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

contract NFTSwap {
    //订单结构体，owner：发起者，price：需要交易的价格
    struct Order {
        address owner;
        uint256 price;
    }

    //多个订单的映射。mapping(key:nftcontract nft合约,value:map(key:tokenid nft产品,value:对应的order结构体))
    mapping(address => mapping(uint256 => Order)) public orders;
    //记录合约对应的nft藏品是否已经挂出的映射。
    //一个合约对应着多个nft藏品mapping(key:nftcontract nft合约,value:map(key:tokenid nft产品,value:bool 是否挂出))
    mapping(address => mapping(uint256 => bool)) public isListed;

    /*
        卖家 挂单的functions：
        nftContract:nft合约；tokenId:nft产品的唯一标识；price：交易价格；
    */
    function list(
        address nftContract,
        uint256 tokenId,
        uint256 price
    ) external {
        require(price > 0, "Price must be greater than 0");//如果要挂出那么价格必须大于0
        require(!isListed[nftContract][tokenId], "Already listed");//检查这个nft合约对应的tokenid的nft数字产品是否已经挂出
        IERC721 nft = IERC721(nftContract);//创建 遵循ERC721标准的IERC721接口变量（对象）命名为nft（nftContract是一个合约地址，将这个合约地址传进去得到的变量遵循IERC721，可以调用方法获取信息）
        require(nft.ownerOf(tokenId) == msg.sender, "You don't own this NFT");//检查 此tokenid的nft产品是否是当前发起交易者所有
        //以上检查都成立 则挂起订单
        orders[nftContract][tokenId] = Order({
            owner: msg.sender,
            price: price
        });
        isListed[nftContract][tokenId] = true;
    }

    /*
        卖家 撤单的function：
        nftContract:nft合约；tokenId:nft产品的唯一标识；
    */
    function revoke(
        address nftContract,
        uint256 tokenId
    ) external {
        require(isListed[nftContract][tokenId], "Not listed");//检查这个nft合约对应的tokenid的nft数字产品是否已经挂出
        require(orders[nftContract][tokenId].owner == msg.sender, "You are not the owner of this order");//（根据nftContract合约地址和nft产品的tokenid，来对应一个订单结构体）检查这个订单的发起人是否是调用的用户
        //以上条件都成立则删除订单集合中的这个订单结构体，挂单的映射置为false。   撤单
        delete orders[nftContract][tokenId];
        isListed[nftContract][tokenId] = false;
    }

    /*
        卖家 修改价格的function：
        nftContract:nft合约；tokenId:nft产品的唯一标识；price：交易价格；
    */
    function update(
        address nftContract,
        uint256 tokenId,
        uint256 newPrice
    ) external {
        require(isListed[nftContract][tokenId], "Not listed");//检查这个nft合约对应的tokenid的nft数字产品是否已经挂出
        require(orders[nftContract][tokenId].owner == msg.sender, "You are not the owner of this order");//（根据nftContract合约地址和nft产品的tokenid，来对应一个订单结构体）检查这个订单的发起人是否是调用的用户
        require(newPrice > 0, "Price must be greater than 0");//如果要挂出那么价格必须大于0
        //以上条件都成立则修改定价
        orders[nftContract][tokenId].price = newPrice;
    }

    /*
        买家 购买nft藏品的function：
        nftContract:nft合约；tokenId:nft产品的唯一标识；
    */
    function purchase(
        address nftContract,
        uint256 tokenId
    ) external payable {
        require(isListed[nftContract][tokenId], "Not listed");//检查这个nft合约对应的tokenid的nft数字产品是否已经挂出
        //如果挂出了 则拿到这个订单信息的结构体
        Order storage order = orders[nftContract][tokenId];
        require(msg.value >= order.price, "Insufficient funds");//检查调用者支付的额度是否大于等于订单中的价格额度
        //以上条件都满足之后，就要进行 遵循ERC721标准的nft藏品的交易
        IERC721 nft = IERC721(nftContract);
        /*
        根据 IERC721 接口的定义
        safeTransferFrom 函数用于安全地将一个 NFT 从一个地址（order.owner，即当前持有 NFT 的卖家地址）转移到另一个地址（msg.sender，即发起购买操作的买家地址）。
        */
        nft.safeTransferFrom(order.owner, msg.sender, tokenId);

        // 将以太币转给之前拥有nft藏品的卖家的账户，先将卖家账户地址转化为可接受以太币的类型payable,使用call函数，gas无限制
        (bool success, ) = payable(order.owner).call{value: order.price}("");
        require(success, "Transfer failed");
        // 交易完成将订单从订单列表中删除
        delete orders[nftContract][tokenId];
        // 挂起的记录改为false
        isListed[nftContract][tokenId] = false;
        // 检查 支付的金额是否超过了订单的金额，如果超过了则将金额转回买家的账户地址
        if (msg.value > order.price) {
            payable(msg.sender).transfer(msg.value - order.price);
        }
    }
}
