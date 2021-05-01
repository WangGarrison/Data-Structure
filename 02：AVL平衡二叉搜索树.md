# AVL平衡二叉搜索树

> AVL树得名于它的发明者G. M. Adelson-Velsky和E. M. Landis
>
> AVL树本质上还是一棵二叉搜索树，它的特点是：
>
> - 本身首先是一棵二叉搜索树。
>
> - 带有平衡条件：每个结点的左右子树的高度之差的绝对值（平衡因子）最多为1。
>
> 也就是说，==AVL树，本质上是带了平衡功能的二叉搜索树==

![image-20210323110701634](img/02%EF%BC%9AAVL%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210323110701634.png)

# 右旋，左旋

![image-20210402200630531](img/02%EF%BC%9AAVL%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210402200630531.png)

**右旋、左旋代码：**

- 结点旋转（右旋为例）：先用child记录node->left，然后将node->left变为child->right，然后将node旋转下去：child->right = node；
- 更新结点高度属性
- 返回新根

```cpp
#include <iostream>
#include <cmath>
using namespace std;

//AVL树
template<typename T>
class AVLTree
{
private:
	//定义AVL树结点类型
	struct Node
	{
		Node(T data = T()) : data_(data), left_(nullptr), right_(nullptr), height_(1) {}
		T data_;
		Node* left_;
		Node* right_;
		int height_; //记录结点的高度值
	};

	//返回结点的高度值
	int height(Node* node)
	{
		return node == nullptr ? 0 : node->height_;
	}

	//右旋转操作，' / '型，以参数node(右上结点)为中心作右旋转操作，并把新的根结点返回
	Node* rightRotate(Node* node)
	{
		//结点旋转
	    Node* child = node->left_;
		node->left_ = child->right_;
		child->right_ = node;

		//更新高度，这时候node已经旋转下去了，child旋转上去了
		node->height_ = max(height(node->left_), height(node->right_)) + 1;
		child->height_ = max(height(child->left_), height(child->right_)) + 1;
		
		//返回旋转后新的根结点
		return child;
	}

	//左旋转操作，' \ '型，以参数node(左上结点)为中心作左旋转操作，并把新的根结点返回
	Node* leftRotate(Node* node)
	{
		//结点旋转
		Node* child = node->right_;
		node->right_ = child->left_;
		child->left_ = node;

		//更新高度，这时候node已经旋转下去了，child旋转上去了
		node->height_ = max(height(node->left_), height(node->right_)) + 1;
		child->height_ = max(height(child->left_), height(child->right_)) + 1;

		//返回旋转后新的根结点
		return child;
	}

	Node* root_;  //指向根节点
};
```

# 左右旋，右左旋

![image-20210323123734730](img/02%EF%BC%9AAVL%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210323123734730.png)

**左右旋、有左旋代码：**

- 左右旋(左平衡)：child先左旋，并且新树接回到原树；node再右旋
- 右左旋(右平衡)：child先右旋，并且新树接回到原树；node再左旋

```cpp
//左右旋(左平衡)：child先左旋，并且新树接回到原树；node再右旋
Node* leftBalance(Node* node)
{
    //child先左旋，并且新树接回到原树
    node->left_ = leftRotate(node->left_);

    //node再右旋
    return rightRotate(node);
}

//右左旋(右平衡)：child先右旋，并且新树接回到原树；node再左旋
Node* rightBalance(Node* node)
{
    //child先右旋，并且新树接回到原树
    node->right_ = rightRotate(node->right_);

    //node再左旋
    return leftRotate(node);
}
```

# AVL树的插入

在BST树插入代码的基础上添加代码

- 在插入完成后回溯的时候，判断是否失衡，若失衡进行旋转调整

- 记得要更新高度

```cpp
public:	
	AVLTree() : root_(nullptr)	 {}

	//AVL树的插入接口
	void Insert(const T& val)
	{
		root_ = Insert(root_, val);
	}

	//AVL树的插入实现
	Node* Insert(Node* node, const T& val)
	{
		if (node == nullptr)//递归结束，找到插入的结点了
		{
			return new Node(val);
		}
		if (val < node->data_)//往左边插
		{
			node->left_ = Insert(node->left_, val);

			//添加1：在递归回溯时，判断结点是否失衡
			if (height(node->left_) - height(node->right_) > 1)//node的左子树太高，node失衡了
			{
				if (height(node->left_->left_) > height(node->left_->right_))
				{
				    //左左太高，右旋
					node = rightRotate(node);
				}
				else
				{
					//左右太高了，左右旋
					node = leftBalance(node);
				}
			}
		}
		else if(val > node->data_)//往右边插
		{
			node->right_ = Insert(node->right_, val);

			//添加2：在递归回溯时，判断结点是否失衡
			if (height(node->right_) - height(node->left_) > 1)//node的右子树太高，node失衡了
			{
				if (height(node->right_->right_) > height(node->right_->left_))
				{
					//右右太高，左旋
					node = leftRotate(node);
				}
				else
				{
					//右左太高了，右左旋
					node = rightBalance(node);
				}
			}
		}
		else//元素相同，不用插入，直接回溯
		{
			;
		}
		//添加3：因为子树中增加了新的结点，在递归回溯时，更新结点高度
		node->height_ = max(height(node->left_), height(node->right_)) + 1;
		return node;
	}
```

# AVL树的删除

相比于BST树的删除，添加了以下代码：

- 左子树删除节点，可能造成右子树太高，进行旋转调节
- 右子树删除节点，可能造成左子树太高，进行旋转调节
- 为了避免删除前驱或者后继节点，造成结点失衡，谁高用谁来覆盖当前结点进行删除
- 记得更新结点高度

```cpp
//AVL树的删除接口
void remove(const T& val)
{
    root_ = remove(root_, val);
}	
//AVL树的删除实现
Node* remove(Node* node, const T& val)
{
    if (node == nullptr)
    {
        return nullptr;
    }
    if (val < node->data_)  //在左子树中继续找要删除的结点
    {
        node->left_ = remove(node->left_, val);
        //左子树删除节点，可能造成右子树太高
        if (height(node->right_) - height(node->left_) > 1)
        {
            if (height(node->right_->right_) >= height(node->right_->left_))//右右太高，左旋
            {
                node = leftRotate(node);
            }
            else//右左太高，右左旋转
            {
                node = rightBalance(node);
            }
        }
    }
    else if(val > node->data_)  //在右子树中继续找要删除的结点
    {
        node->right_ = remove(node->right_, val);
        //右子树删除节点，可能造成左子树太高
        if (height(node->left_) - height(node->right_) > 1)
        {
            if (height(node->left_->left_) >= height(node->left_->right_))//左左太高，右旋
            {
                node = rightRotate(node);
            }
            else//左右太高，左右旋转
            {
                node = leftBalance(node);
            }
        }
    }
    else  //找到了要删除的结点
    {
        //先处理有两个孩子的删除情况
        if (node->left_ != nullptr && node->right_ != nullptr)
        {
            //为了避免删除前驱或者后继节点，造成结点失衡，谁高用谁来覆盖当前结点进行删除
            if (height(node->left_) >= height(node->right_))
            {
                //用前驱替换
                Node* pre = node->left_;
                while (pre->right_ != nullptr)
                {
                    pre = pre->right_;
                }
                node->data_ = pre->data_;
                node->left_ = remove(node->left_, pre->data_);//删除前驱
            }
            else
            {
                //用后继替换
                Node* post = node->right_;
                while (post->left_ != nullptr)
                {
                    post = post->left_;
                }
                node->data_ = post->data_;
                node->right_ = remove(node->right_, post->data_); //删除后继
            }
        }
        else  //删除的结点最多有一个孩子
        {
            if (node->left_ != nullptr)
            {
                Node* child = node->left_;
                delete node;
                return child;
            }
            else if (node->right_ != nullptr)
            {
                Node* child = node->right_;
                delete node;
                return child;
            }
            else//删除的结点没有孩子
            {
                return nullptr;
            }
        }
    }
    //更新结点高度
    node->height_ = max(height(node->left_), height(node->right_)) + 1;

    return node;  //递归回溯过程中，把当前结点给父结点返回
}
```

# AVL树的查询

由于AVL树的查询并不会影响树的高度，所以查询和BST树是一样的，见BST博客里的代码

# AVL树优缺点

优点：==它是平衡的二叉搜索树：查询效率高O(lgn)==

缺点：执行插入还是删除操作，只要不满足平衡的条件，就要通过旋转来保持平衡，而旋转是非常耗时的，插入操作最多旋转次数2，删除操作最多旋转次数O(lgn)

由此我们可以知道**AVL树适合用于插入与删除次数比较少，但查找多的情况**

由于维护这种高度平衡所付出的代价比从中获得的效率收益还大，故而实际的应用不多，更多的地方是用追求局部而不是非常严格整体平衡的红黑树。当然，**如果应用场景中对插入删除不频繁，只是对查找要求较高，那么AVL还是较优于红黑树。**

# 有parent域的AVL树的左旋、右旋

![image-20210410173149429](img/02%EF%BC%9AAVL%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210410173149429.png)