## 1.1 B树节点添加过程图
![1](https://github.com/jeanter/H2ResearchNote/raw/main/btree_add15.jpg)


## 1.2 B树节点删除过程图
![1](https://github.com/jeanter/H2ResearchNote/raw/main/btree.jpg)

## 1.2 B树删除叶子结点 过程图

             case REMOVE: {
                        if (index < 0) {
                            if(!locked && rootReference != getRoot()) {
                                decisionMaker.reset();
                                continue;
                            }
                            return null;
                        }
                         //p 此时是叶子节点 ,p.getTotalCount() 就是 p.keys.length  
                        // 分支1 **********************// 当前节点就一个元素
                        if (p.getTotalCount() == 1 && pos != null) {
                        	System.out.println("remove key "+ key +"  p.getTotalCount() == 1 " +" current p info :"+ PageUtil.getPageInfo(p));
                            int keyCount;
                            do {
                            	// p现在为p的父节点
                                p = pos.page;
                                index = pos.index;
                                // pos 继续上升一层
                                pos = pos.parent;
                                // 此时已经是父节点的keyCount 
                                keyCount = p.getKeyCount();
                                // condition below should always be false, but older
                                // versions (up to 1.4.197) may create
                                // single-childed (with no keys) internal nodes,
                                // which we skip here
                            } while (keyCount == 0 && pos != null);
                            // 父节点也刚好只有一个key p 又变成了 leafPage
                            if (keyCount <= 1) {
                            	System.out.println("remove key keyCount <= 1   "+ key  +",keyCount : "+keyCount + "; current p info :"+ PageUtil.getPageInfo(p));
                                if (keyCount == 1) {
                                    assert index <= 1;
                                    //子节点变成了父节点
                                    p = p.getChildPage(1 - index);
                                    System.out.println(" remove key no remove "+ key + " ,current p info :"+ PageUtil.getPageInfo( p ));
                                } else {
                                    // if root happens to be such single-childed
                                    // (with no keys) internal node, then just
                                    // replace it with empty leaf
                                    p = Page.createEmptyLeaf(this);
                                    System.out.println(" remove key else no remove "+ key + " ,current p info :"+ PageUtil.getPageInfo( p ));
                                }
                                break;
                            }
                        }
                        // 如果分支1成立，p此时是leaf的父节点. 调用NonLeaf.remove(index)  先remove key,再remove child
                        System.out.println("start***** remove key "+ key + " ,current p info :"+ PageUtil.getPageInfo( p ));
                        p = p.copy();
                        p.remove(index);
                        System.out.println("end***** remove key "+ key + " ,current p info :"+ PageUtil.getPageInfo( p ));
                        break;

remove key 5  p.getTotalCount() == 1  current p info :page id: 776,page_type: org.h2.mvstore.Page$Leaf;keys: [5]  <br />
start***** remove key 5 ,current p info :page id: 283,page_type: org.h2.mvstore.Page$NonLeaf;keys: [3, 5];children id: [832,560,776,] <br />
end***** remove key 5 ,current p info :page id: 0,page_type: org.h2.mvstore.Page$NonLeaf;keys: [3];children id: [832,560,] <br />
### 便于图形观察，将h2的keysPerPage设置为4，即每个page4个key 
### 删除一个叶子节点 叶子节点只有一个key,先删除叶子节点(在父节点的child[]中删除)，并且一并删除父节点对应的key  <br />
![](https://github.com/jeanter/H2ResearchNote/blob/main/btree_del5_1.jpg)

### 删除当前叶子节点只有一个元素，并且父节点只有一个元素，（两个child），这个时候，父节点的另一个child节点变成当前的父节点. <br />
remove key 3  p.getTotalCount() == 1  current p info :page id: 024,page_type: org.h2.mvstore.Page$Leaf;keys: [3] <br />
remove key keyCount <= 1   3,keyCount : 1; current p info :page id: 169,page_type: org.h2.mvstore.Page$NonLeaf;keys: [3];children id: [832,024,]<br />
 remove key no remove 3 ,current p info :page id: 832,page_type: org.h2.mvstore.Page$Leaf;keys: [1, 2]<br />
start replacePage path.index:0; path.page: page id: 313,page_type: org.h2.mvstore.Page$NonLeaf;keys: [7];children id: [169,021,]; page   :page id: 832,page_type: org.h2.mvstore.Page$Leaf;keys: [1, 2]<br />
replacePage index:0; replacement: page id: 0,page_type: org.h2.mvstore.Page$NonLeaf;keys: [7];children id: [169,021,]; child   :page id: 832,page_type: org.h2.mvstore.Page$Leaf;keys: [1, 2]<br />
end replacePage  replacement: page id: 0,page_type: org.h2.mvstore.Page$NonLeaf;keys: [7];children id: [832,021,]<br />
end remove key 3 ,replacePage************  current p info :page id: 832,page_type: org.h2.mvstore.Page$Leaf;keys: [1, 2]<br />
![](https://github.com/jeanter/H2ResearchNote/blob/main/btree_del3.jpg)

remove key 1  p.getTotalCount() == 1  current p info :page id: 056,page_type: org.h2.mvstore.Page$Leaf;keys: [1]<br />
remove key keyCount <= 1   1,keyCount : 1; current p info :page id: 201,page_type: org.h2.mvstore.Page$NonLeaf;keys: [7];children id: [056,021,]<br />
remove key no remove 1 ,current p info :page id: 021,page_type: org.h2.mvstore.Page$NonLeaf;keys: [9, 11, 13];children id: [152,880,672,466,]<br />
end replacePage  replacement: page id: 021,page_type: org.h2.mvstore.Page$NonLeaf;keys: [9, 11, 13];children id: [152,880,672,466,]<br />
end remove key 1 ,replacePage************  current p info :page id: 021,page_type: org.h2.mvstore.Page$NonLeaf;keys: [9, 11, 13];children id: [152,880,672,466,]<br />
page id: 021,page_type: org.h2.mvstore.Page$NonLeaf;keys: [9, 11, 13];children id: [152,880,672,466,]<br />
page id: 152,page_type: org.h2.mvstore.Page$Leaf;keys: [7, 8]<br />
page id: 880,page_type: org.h2.mvstore.Page$Leaf;keys: [9, 10]<br />
page id: 672,page_type: org.h2.mvstore.Page$Leaf;keys: [11, 12]<br />
page id: 466,page_type: org.h2.mvstore.Page$Leaf;keys: [13, 14, 15, 16]<br />
### 删除key 1后 根节点的最后一个叶子节点变成跟节点 <br />
![](https://github.com/jeanter/H2ResearchNote/blob/main/btree_del7.jpg)

### 删除key 1后  ，再添加1 
![](https://github.com/jeanter/H2ResearchNote/blob/main/breee_add1.jpg)


## 1.3 H2 CursorPos CursorPos.traverseDown(rootPage, key)

        CursorPos<K,V> pos = CursorPos.traverseDown(rootPage, key);
      /** 此方法通过rootPage查找到指定key所在的leafPage,每一次进入到下个节点，通过CursorPos保持父节点和当前节点，
          子节点的index,从而保存整个查找路径上的全部节点路径信息
  
     * Searches for a given key and creates a breadcrumb trail through a B-tree
     * rooted at a given Page. Resulting path starts at "insertion point" for a
     * given key and goes back to the root.
              * 搜索给定的键并通过 B 树创建breadcrumb路径
              *从根节点到指定key所在的page。 结果路径从“插入点”开始
              * 给定key并返回到根。
      * * Instance代表链表中的一个节点，它追踪路径
                     * 从叶节点中的特定（目标）键一直到根
                   *（自下而上的路径）。
     *
     * @param page      root of the tree
     * @param key       the key to search for
     * @return head of the CursorPos chain (insertion point)
     */
    static <K,V> CursorPos<K,V> traverseDown(Page<K,V> page, K key) {
        CursorPos<K,V> cursorPos = null;
        while (!page.isLeaf()) {
            int index = page.binarySearch(key) + 1;
            if (index < 0) {
                index = -index;
            }
            //  三个参数  page表示当前到达的，index ,index表示下一级节点在page叶子节点的索引, cursorPos表示父节点
            cursorPos = new CursorPos<>(page, index, cursorPos);
            page = page.getChildPage(index);
        }
        return new CursorPos<>(page, page.binarySearch(key), cursorPos);
    }
