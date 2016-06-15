
## PaypalIPN和PDT变量表

### IPN 和 PDT 变量:买家信息

| 变量名 | 可能的值 | 描述 | 长度 |
|---|---|---|---|
| address_city |  | 客户地址中的市/县 | 40 |
| address_country  |  | 客户地址中的国家或地区 | 64 |
| address_country_code |  | 两位 ISO 3166 国家或地区代码 | 2 |
| address_name  |  | 用于地址的名称(在客户提供礼品地址时包含在内) | 128 |
| address_state |  | 客户地址中的省/直辖市/自治区。 | 40 |
| address_status | Confirmed / Unconfirmed | 客户提供的是否是已确认的地址 |  |
| address_street  |  | 客户的街道地址 | 200 |
| address_zip  |  | 客户地址中的邮政编码 | 20 |
| first_name |  | 客户的名 | 64 |
| last_name  |  | 客户的姓 | 64 |
| payer_business_name |  | 客户的公司名称,如果客户代表企业 | 127 |
| payer_email  |  | 客户的主要邮件地址, 使用该电子邮件提供所有信用记录 | 127 |
| payer_id |  | 唯一客户号 | 13 |
| payer_status | verified / unverified | 客户是否是已认证的 PayPal 账户, 用户通过 paypal 账户关联到到信用卡即可通过认证 |  |
| residence_country |  | 两位 ISO 3166 国家或地区代码 | 2 |

### IPN 和 PDT 变量:基本信息

| 变量名 | 描述 | 长度 |
|---|---|---|
| business | 收款人(即商家)的电子邮件地址或账户号 如果付款发送至主账户,则等于receiver_email,尤其指“网站付款”按钮 HTML 代码中传递的 business变量的返回值 | 127 |
|  item_name | 由您(商家)传递的物品名称, 如果不是由您传递,则由您的客户输入, 如果是购物车交易,PayPal 将附加物品号(例如, item_name1 、 item_name2 ) | 127 |
| item_number | 您用于跟踪购买的传递变量。在付款完成时,它会传回给您。如果省略, 则将没有变量传回给您。 | 127 |
| quantity | 由您的客户输入或由您(商家)传递的数量 如果是购物车交易,PayPal 将附加物品号(例如,quantity1、quantity2) |  |
| receiver_email | 收款人(即商家)的主要邮件地址,如果付款不是发送到 PayPal 账户上的主要邮件地址,则 receiver_email 依旧是主要邮件地址 | 127 |
| receiver_id | 收款人(即商家)的唯一账户号。这与收款人的推荐号相同 | 13 |

### IPN 和 PDT 变量:高级及自定义信息

| 变量名 | 描述 | 长度 |
|---|---|---|
| custom | 由您(商家)传递的自定义值。在任何情形下,都不会向您的客户显示 这些传递变量 | 255 |
|  invoice | 可供您用来识别此次购物的帐单号码的转递变量, 如果省略,则没有变量传回 | 127 |
|  memo | 您的客户在 PayPal 网站付款提示栏中输入的备忘信息 | 255 |
| option_name1 | 选项 1 名称(由您申请) | 64 |
|  option_name2 | 选项 2 名称(由您申请) |  |
|  option_selection1 | 选项 1 选择(由客户输入) | 200 |
|  option_selection2 | 选项 2 选择(由客户输入) |  |
|  tax | 对付款收取的税费金额 | 2 |

### IPN 和 PDT 变量:购物车信息

<table>
  <tbody>
    <tr>
      <td>
      <p>变量名</p>
      </td>
      <td>
      <p>可能的值</p>
      </td>
      <td>
      <p>描述</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_gross_x</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>所示金额所用币种为 mc_currency ,其中 x 为 购物车明细物品号。mc_gross_x 总和应等于 mc_gross</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_handling_x</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>x 代表购物车明细物品号。由于 mc_handling 变 量中还包含 handling_cart 整个购物车范围内的 网站付款变量,因此 mc_handling_x 总和不一定 等于 mc_handling</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>num_cart_items</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>如果此交易是 PayPal 购物车交易,则为购物车 中的物品数</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>option_name1</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 将附加物品号,其中 x 代表购物车明细 物品号(例如,option_name1、option_name2)</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>option_name2</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 将附加物品号,其中 x 代表购物车明细 物品号(例如,option_name2、option_name2)</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>option_selection1_x</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 将附加物品号(例如,option_selection1 、 option_selection2),其中 x 代表购物车明细物品 号</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>option_selection2_x</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 将附加物品号,其中 x 代表购物车明细 物 品 号 ( 例 如 , option_selection1 、 option_selection2)</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>parent_txn_id</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>在退款、撤销或取消撤销的情况下,该变量包 含原定交易的 txn_id,而 txn_id 包含新交易的新识别号 字符长度和限制:17</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>payment_date</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 生成的时间/日期戳记 格式:&ldquo; 18:30:30 Jan 1, 2000 PST &rdquo;</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>payment_status</p>
      </td>
      <td>
      <p>Canceled-Reversal</p>
      <p>Completed</p>
      <p>Denied</p>
      <p>Expired</p>
      <p>Failed</p>
      <p>In-Progress</p>
      <p>Partially-Refunded</p>
      <p>Pending</p>
      <p>Processed</p>
      <p>Refunded</p>
      <p>Reversed</p>
      <p>Voided</p>
      </td>
      <td>
      <p>Canceled-Reversal:这意味着已经取消了撤销。 例如,您在与客户的争议中获胜,先前撤销的 交易资金已退回给您 Completed:付款已完成,资金已成功增加到您 的账户余额中 Denied:您拒绝了付款。只有该款项此前因。 PendingReason 元素说明的可能原因而待付时, 才会发生此类情况 Expired:这个授权已经过期,无法捕获 Failed:付款失败。只有当付款来自于客户的银 行账户时,才会发生此类情况 In-Progress:这笔交易处于授权认证中。 Partially-Refunded:这笔交易被部分退款 Pending:款项待付,请查看 PendingReason 了 解更多信息Refunded:您退还了付款 Reversed:付款由于扣款索偿或其他撤销类型而 撤销。资金已从您的账户余额中扣除,并已退 还给买家。reason_code 变量指明了撤销原因 Processed:付款已被接受 Voided:此授权无效&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>payment_type</p>
      </td>
      <td>
      <p>echeck instant</p>
      </td>
      <td>
      <p>echeck:该款项通过电子支票支付 instant:该项付款通过 PayPal 余额、信用卡或 即时转帐支付</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>pending_reason</p>
      </td>
      <td>
      <p>address authorization echeck<br /> intl <br /> multi-currency<br /> unilateral upgrade verify other</p>
      </td>
      <td>
      <p>只有在payment_status=Pending时,才会设置此 变量 address:款项待付,原因是客户未提供已确认 的送货地址,而您的收款习惯设定设为允许手 动接受或拒绝每笔此类付款。若要更改习惯设定,请前往您的用户信息中的习惯设定部分 authorization:您在 SetExpressCheckoutRequest 上 设 置 了 &lt;PaymentAction&gt; Authorization&lt;/PaymentAction&gt; ,而尚未获取资 金 echeck:款项待付,原因是其通过电子支票付 款,而电子支票尚未结清 intl:款项待付,原因是您持有非美国账户,且 没有提现机制。您必须在账户信息中手动接受 或拒绝该笔付款 multi-currency:您在发送的货币中没有余额, 并且未将收款习惯设定设为自动兑换和接受付 款。您必须手动接受或拒绝该笔付款 unilateral:款项待付,原因是付款的接收电子 邮件地址尚未注册或确认 upgrade:款项待付,原因是其通过信用卡付款, 因此您必须将账户升级为企业账户或高级账户 状态,方可接收资金。upgrade 也可能表示您已 达到账户的月交易限额 verify:款项待付,原因是您尚未经过认证。您 必须先认证您的账户,才能接受该笔付款 other:款项待付,原因非以上所列各项。若要 了解更多信息,请与贝宝客户服务联系。</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>reason_code</p>
      </td>
      <td>
      <p>chargeback guarantee buyer-complaint refund <br /> other</p>
      </td>
      <td>
      <p>只有在 payment_status = Reversed 或 Refunded 时,才会设置此变量 chargeback:由于客户提出扣款索偿,因此撤销 这笔交易 guarantee:由于客户触发退款担保,因此撤销 这笔交易 buyer-complaint:由于客户就交易提出投诉,因 此撤销这笔交易 refund:由于您向客户退款,因此撤销这笔交易 other:由于上述原因以外的其他原因,撤销这笔交易。</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>tax</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 将附加物品号(例如,item_name1、 item_name2 )。只有对个别购物车物品收取特 定税费时,才会在其中包含 tax_x 变量。由于可 能对购物车中其他物品收取用户信息税费,因 此 tax_x 的总和不一定等于 tax</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>txn_id</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>PayPal 系统生成的唯一交易号 字符长度和限制:17</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>txn_type</p>
      </td>
      <td>
      <p>cart</p>
      <p>express_checkout</p>
      <p>merch_pmt</p>
      <p>send_money</p>
      <p>virtual_terminal</p>
      <p>web-accept</p>
      </td>
      <td>
      <p>cart:交易由客户通过&ldquo;PayPal 购物车&rdquo;功能创建 send-money:交易由客户从 PayPal 网站上的付 款选项卡中创建 web-accept:交易由客户通过&ldquo;立即购买&rdquo;、&ldquo;捐 赠&rdquo;或&ldquo;竞拍&rdquo;智能标识创建。</p>
      <p>&nbsp;</p>
      </td>
    </tr>
  </tbody>
</table>

### IPN 和 PDT 变量:货币及货币兑换信息

<table>
  <tbody>
    <tr>
      <td>
      <p>变量名</p>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>可能的值</p>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>描述</p>
      <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_currency</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>付款货币</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_fee</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>与付款关联的交易费。mc_gross 减去 mc_fee 等于存 入 receiver_email 账户的金额。等于美元付款 payment_fee 如果该金额为负,则表示退款或撤销,原定交易费的 全部或部分金额都可以是这两种付款状态之一</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_gross</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>&nbsp;&nbsp; 扣除交易费之前的客户付款全部金额。等于美元付款</p>
      <p>payment_gross</p>
      <p>如果该金额为负,则表示退款或撤销,原定交易费的 全部或部分金额都可以是这两种付款状态之一</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_handling</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>这是与交易相关的手续费总额</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>mc_shipping</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>这是与交易相关的运费总额</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>payment_fee</p>
      </td>
      <td>
      <p>&nbsp;</p>
      </td>
      <td>
      <p>与付款相关的美元交易费。payment_gross 减去 payment_fee 等于存入收款人电子邮件账户的金额。 对于非美元付款为空。这个旧字段已由 mc_fee 取代 如果该金额为负,则表示退款或撤销,原定交易费的 全部或部分金额都可以是这两种付款状态之一</p>
      </td>
    </tr>
    <tr>
      <td>
      <p>payment_gross</p>
      </td>
      <td>
      <p>&nbsp;&nbsp; &nbsp; &nbsp;</p>
      </td>
      <td>
      <p>&nbsp; &nbsp; 扣除交易费之前的客户付款全部美元金额。对于非美元付款将为空。这个旧字段已由mc_gross 取代如果该金额为负,则表示退款或撤销,原定交易费的全部或部分金额都可以是这两种付款状态之一</p>
      </td>
    </tr>
  </tbody>
</table>
