1. UI 渲染中间， 如果插入半透RT， 半透RT 在渲染时 alpha 的blend 方式和color 不应该相同。
套路就是：
*  针对所有不透明物件和所有半透物件、以及 Alpha Test 物件（通过 Alpha Test 的片元保留原 alpha 值），可以采用通用的做法：
src_blend_alpha_factor = 1 - dest_alpha，dest_blend_alpha_factor = 1
src_blend_factor = src_alpha      dest_blend_factor = 1-src_alpha

* RT 绘制回BackBuffer 的时候， src_blend_factor = 1  dest_blend_factor = 1-src_alpha

可以这样理解：
  如果按照排序好的半透混合方式：颜色值 c， alpha 值 用字母a
两个像素 (c0, a0), (c1, a1)混合的情况：
最终的颜色为： c0*(1-a1）+ c1*a1
如果是4个像素 (c0, a0), (c1, a1), (c2, a2), (c3, a3)：
 最终颜色为：（(((c0*(1-a1) + c1*a1)*(1-a2)+c2*a2)  * (1-a3) + c3*a3)
现在考虑， 提前先混合 c1，c2, c3 , 然后用混合后的结果和c1 再混合, 这种情况下， 可以将上面的公式展开
c0*(1-a1)*(1-a2)*(1-a3) + ((c1*a1(1-a2) + c2*a2)*(1-a3) + c3*a3
=c0*(1-a1)*(1-a2)*(1-a3) + (((0*(1-a1)+ c1*a1)(1-a2) + c2*a2)*(1-a3) + c3*a3
后面加粗的部分就是正常的颜色混合， 将c0 和 RT 的颜色进行混合时， 需要alpha 值 (1-a1)*(1-a2)*(1-a3), 这部分可以在RT 的alpah通道中。
 如果直接存储  (1-a1)*(1-a2)*(1-a3) 那么绘制过程中半透明的blend 设置应该为 src_blend_alpha_factor =0  dst_blend_alpha_factor =1-src_alpha;
考虑初始状态， 要想获得 (1-a1)  那么RT buffer 的初始值必须是1 。
这里存在一个小技巧:
如果在RT 中存储 1-(1-a1)*(1-a2)*(1-a3) ；  src_blend_alpha_factor =1-dest_alpha  dst_blend_alpha_factor =1;
初始状态下RT alpha通道为0 即可的。
