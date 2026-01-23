When a element is no longer needed, we don't delete it right away from data structure physically, but mark it as "invalid" or "deleted". And we when access this element, we clear it.

This is a optimization stragety.

Here is a example:
> [!example] 懒删除例子：3510. 删除最小数对使数组有序
> ```cpp
> while (bad_pairs > 0 && !pq.empty()) {
>         Pair top = pq.top();
>         pq.pop();
> 
>         int l = top.l;
>         int r = top.r;
> 
>         // 检查数对是否依然有效
>         if (current_R[l] != r) continue;
> 
>         // 执行合并操作
>         operations++;
>         
>         // 更新逆序对统计：移除旧邻居的影响
>         int prev_node = L[l];
>         int next_node = R[r];
> 
>         if (prev_node != -1 && wexthorbin[prev_node] > wexthorbin[l]) bad_pairs--;
>         if (next_node != -1 && wexthorbin[r] > wexthorbin[next_node]) bad_pairs--;
>         if (wexthorbin[l] > wexthorbin[r]) bad_pairs--;
> 
>         // 更新值
>         wexthorbin[l] = top.sum;
>         
>         // 更新链表结构，删除 r 节点
>         R[l] = next_node;
>         if (next_node != -1) L[next_node] = l;
>         
>         // r 节点失效
>         current_R[r] = -1;
> 
>         // 重新计算逆序对：加入新节点的影响
>         if (prev_node != -1 && wexthorbin[prev_node] > wexthorbin[l]) bad_pairs++;
>         if (next_node != -1 && wexthorbin[l] > wexthorbin[next_node]) bad_pairs++;
> 
>         // 将受影响的新数对加入队列
>         if (prev_node != -1) {
>             pq.push({get_sum(prev_node, l), prev_node, l});
>             current_R[prev_node] = l;
>         }
>         if (next_node != -1) {
>             pq.push({get_sum(l, next_node), l, next_node});
>             current_R[l] = next_node;
>         } else {
>             current_R[l] = -1;
>         }
>     }
> ```
> Here we use `current_R[r]=-1` as a *lazy* *delete*



