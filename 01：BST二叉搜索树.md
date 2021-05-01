# BST二叉搜索树

> BST树：Binary Search/Sort Tree，二叉搜索/排序树
>
> 对于二叉树上的每一个结点，如果满足<font color='red'>**左孩子的值 < 父结点的值 < 右孩子的值**</font>，并且每一个左子树其所有结点都小于其根，右子树所有结点都大于其根，则把该二叉树称作BST树

二分查找算法其实就是在遍历二叉搜索树（BST树），对二叉搜索树进行==中序遍历==可以得到一个 递增序列

在有序序列里面的==二分折半查找就相当于BST树从根节点沿着某一个路径进行查找==，就相当于走了个树的高度O(lgn)

![image-20210318201710290](img/01%E5%A4%8D%E6%9D%82%E5%BA%A6.img/image-20210318201710290.png)

# BST树的插入

#### 非递归解法

![image-20210318205615572](img/01%E5%A4%8D%E6%9D%82%E5%BA%A6.img/image-20210318205615572.png)

```cpp
#include <functional>
using namespace std;

// BST树代码实现
template<typename T, typename Comp = less<T>>
class BSTree
{
private:
    //结点定义
	struct Node
	{
		Node(T data = T()) :data_(data), left_(nullptr), right_(nullptr) {}
		T data_; 	  //数据域
		Node *left_;  //左孩子域
		Node *right_; //右孩子域
	};
	Node *root_; //指向BST树的根节点
	Comp comp_;  //定义一个函数对象，自定义比大比小，默认是less<
    
public:
	//初始化根节点和函数对象
	BSTree(Comp comp = Comp()) :root_(nullptr), comp_(comp) {}

	//非递归插入操作
	void n_insert(const T &val)
	{
		//树为空，生成根节点
		if (root_ == nullptr)
		{
			root_ = new Node(val);
			return;
		}

		//搜索合适的插入位置，记录父节点的位置
		Node *parent = nullptr;
		Node *cur = root_;
		while (cur != nullptr)
		{
			if (cur->data_ == val)
			{
				//规定不插入元素相同的值
				return;
			}
			else if (comp_(cur->data_, val))//cur->data < val
			{
				parent = cur;
				cur = cur->right_;
			}
			else  //cur->data > val
			{
				parent = cur;
				cur = cur->left_;
			}
		}

		//把新节点插入到parent节点的孩子上
		if (comp_(val, parent->data_)) //val < parent->data
		{
			parent->left_ = new Node(val);
		}
		else
		{
			parent->right_ = new Node(val);
		}
	}
};
```

测试：

```cpp
int main()
{
	BSTree<int> b;
	int arr[] = { 58,24, 67,0,34,62,69,5,41,64,78};
	for (auto it : arr)
	{
		b.n_insert(it);
	}
	b.n_insert(12);
}
```

#### 递归解法

![image-20210319142619423](img/01BST%E6%A0%91.img/image-20210319142619423.png)

```cpp
//递归插入操作，以下代码写在BSTree类内
Node* insert(Node *node, const T & val)
{
    if(node == nullptr)//递归结束，找到插入val的位置，生成新节点并返回其结点地址
    {
        return new Node(val);
    }
    
    if(node->data_ == val)//有相同的值不进行插入
    {
        return node; 
    }
    else if(comp_(node->data_, val))//node->data_ < val，往右子树继续找插入的
    {
        node->right_ = insert(node->right_, val); //插入成功会返回插入的结点，接到插入节点父结点（即node）的右孩子指针
    }
    else//node->data_ > val，往左子树继续找插入的
    {
        node->left_ = insert(node->left_, val); 
    }
    return node;  
}

void insert(const T &val)
{
    root_ = insert(root_, val);
}
```

> 相关题目：
>
> [二叉搜索树中的插入操作](# 二叉搜索树中的插入操作)

# BST树的删除

#### 非递归解法 

![image-20210318214844289](img/01BST%E6%A0%91.img/image-20210318214844289.png)

```cpp
//非递归删除操作，以下代码写在BSTree类内
void n_remove(const T & val)
{
    //树空直接返回
    if (root_ == nullptr)		return;

    //先找到待删结点
    Node *parent = nullptr;
    Node *cur = root_;
    while (cur != nullptr)
    {
        if (comp_(val, cur->data_)) // val < cur->data
        {
            parent = cur;
            cur = cur->left_;
        }
        else if (comp_(cur->data_, val))  //cur->data<val
        {
            parent = cur;
            cur = cur->right_;
        }
        else //cur->data == val，找到待删除的节点
        {
            break;
        }
    }//cur就是要删除的节点

    if (cur == nullptr)		return; //没找到待删除的节点

    //如果是情况3：删除的节点的左右孩子都不是空，找前驱节点
    if (cur->left_ != nullptr && cur->right_ != nullptr)
    {
        //找前驱节点
        parent = cur;
        Node *pre = cur->left_;
        while (pre->right_ != nullptr)
        {
            parent = pre;
            pre = pre->right_;
        }
        //用前驱节点的值覆盖待删除节点的值
        cur->data_ = pre->data_;
        cur = pre;  //这句使得删除pre节点变成删除cur，方便转换成情况1,2进行统一处理
    }

    //统一处理情况1, 2与3的转换：现在cur指向删除节点，parent指向其父结点，统一处理cur指向的节点
    
    Node *child = cur->left_;
    if (child == nullptr)//若有一个孩子，让child指向那个孩子，若左右没孩子，child指向空，方便统一处理：孩子写进父地址域
    {
        child = cur->right_;
    }

    if (parent == nullptr)  //特殊情况：删除的是根节点
    {
        root_ = child;
    }
    else
    {
        //把待删除节点的孩子(不空或nullptr)写进父节点的地址域
        if (parent->left_ == cur)
        {
            parent->left_ = child;
        }
        else
        {
            parent->right_ = child;
        }
    }
    delete cur; //delete要删除的节点
}
```

#### 递归解法

总体思路与非递归解法相同：

- 先找要删除的结点，找到了之后看它有几个孩子
- 情况1,2：只有1个孩子/没有孩子，删除当前结点，并把其孩子返回，接到原树上
- 情况3：有两个孩子，找前驱，用前驱覆盖，再删除前驱

![image-20210320125209477](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A0%E6%A0%91.img/image-20210320125209477.png)

```cpp
//递归删除操作，以下代码写在BSTree类内
Node* remove(Node* node, const T & val)
{
    if(node == nullptr)
    {
        return nullptr;
	}
    
    if(comp_(node->data_, val))//在右子树中继续找val去删除，找到之后更新父结点的右指针域
    {
        node->right_ = remove(node->right_, val);
	}
    else if(comp_(val, node->data_))
    {
        node->left_ = remove(node->left_, val);
	}
    else if(node->data_ == val)//找到了待删除结点
    {
        //情况3：有两个孩子，找前驱节点
        if(node->left_ != nullptr && node->right_ != nullptr)
        {
            //找前驱结点
            Node * pre = node->left_;
            while(pre->right_ != nullptr)
            {
                pre = pre->right_;
            }
            //用前驱结点的值覆盖掉当前结点的值，然后直接递归删除前驱结点就可以了
            node->data_ = pre->data_;
            node->left_ = remove(node->left_, pre->data_);
        }
        //情况1,2，删除掉当前结点，并把孩子结点返回
        else
        {
            Node * child = node->left_ != nullptr ?  node->left_ : node->right_;
            delete node;
            return child;
		}
        
    }
    return node;// 删除完了，把当前结点返回给父结点
}

void remove(const T & val)
{
    root_ = remove(root_, val);  //返回值要更新一下root，因为如果删除的是根节点，新根要指向其孩子  
}
```

> [删除二叉搜索树中的结点](#删除二叉搜索树中的结点)

# BST树的查询

#### 非递归解法

- val与每个结点进行比较，val比当前结点大，就在右边继续找，val比当前结点小，就在左边继续找

```cpp
//非递归查询操作，查询BST树中是否有值为val的结点，以下代码写在BSTree类内
bool n_query(const T & val)
{
    Node* cur = root_;
    while (cur != nullptr)
    {
        if (cur->data_ == val)
        {
            return true;
        }
        else if (comp_(cur->data_, val))
        {
            cur = cur->right_;
        }
        else
        {
            cur = cur->left_;
        }
    }
    return false;
}
```

#### 递归解法

- val与每个结点进行比较，val比当前结点大，就在右边继续找，val比当前结点小，就在左边继续找

```cpp
//递归查询操作，查询BST树中是否有值为val的结点，以下代码写在BSTree类内
Node* query(Node* node, const T & val)
{
    if(node == nullptr)//没找到
    {
        return nullptr;
	}
    if(node->data_ == val)
    {
        return node;
    }
    else if(comp_(node->data_, val))
    {
        return query(node->right_, val);
    }
    else
    {
        return query(node->left_, val);
	}
}
bool query(const T & val)
{
    return nullptr != query(root_, val);
}
```

> [二叉搜索树中的搜索](#二叉搜索树中的搜索)

# BST树前中后序遍历

#### 递归解法

![image-20210319115004796](img/01BST%E6%A0%91.img/image-20210319115004796.png)

上图右上红框里的代码框架，可以把二叉树的每一个结点都遍历到，以下的很多遍历都是基于该框架的。（每一个函数栈就相当于一个结点）

**前、中、后序遍历代码（递归实现）**

- 前序遍历：先访问根节点，再前序遍历左子树，再前序遍历右子树
- 中序遍历：左，根，右
- 后续遍历：左，右，根


```cpp
//前序遍历（递归），以下代码写在BSTree类内
void preOrder(Node * node)
{
    if (node != nullptr)
    {
        cout << node->data_ << " ";  //根
        preOrder(node->left_);       //左
        preOrder(node->right_);      //右
    }
}
void preOrder() 
{
    preOrder(root_);
    cout << endl;
}

//中序遍历（递归）
void inOrder(Node * node)
{
    if (node != nullptr)
    {
        inOrder(node->left_);        //左
        cout << node->data_ << " ";  //根
        inOrder(node->right_);       //右
    }
}
void inOrder()
{
    inOrder(root_);
    cout << endl;
}

//后序遍历（递归）
void postOrder(Node * node)
{
    if (node != nullptr)
    {
        postOrder(node->left_);      //左
        postOrder(node->right_);     //右
        cout << node->data_ << " ";  //根
    }
}
void postOrder()
{
    postOrder(root_);
    cout << endl;
}
```

**仿真指针存储的时候，先序遍历：**

```cpp
//先序遍历（仿真指针存时）：根i，左2i+1，右2i+2
void PreOrder(vector<int> & arr, int i)
{
	if (arr.empty() || i >=arr.size())	return;

	if(arr[i] != -1)	cout << arr[i] << endl;
	PreOrder(arr, 2*i + 1);
	PreOrder(arr, 2 * i + 2);
}
int main()
{
	//一棵树的仿指针存储形式
	vector<int> arr = { 7,8,23,9,15,-1,18,-1,-1,10,20 }; //-1表示空指针
	PreOrder(arr, 0);
}
```

#### 非递归解法（***）

**前序遍历**

- 根左右  V  L  R

- 根节点入栈，栈不为空，取出栈顶并打印，将其右孩子入栈（==后访问的先入栈==），左孩子入栈，栈顶（左孩子）出栈打印，左孩子的右孩子入栈，左孩子的左孩子入栈，栈顶（左左）打印，左左右入栈，左左左入栈，栈顶（左左左）出栈打印......

- 即每次都要用一个栈来保存当前结点的右孩子，这样遍历完左边才能找到右边继续遍历

  <img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A0%E6%A0%91.img/image-20210320143337013.png" alt="image-20210320143337013" style="zoom:33%;" />

```cpp
//非递归前序遍历  VLR
void n_preOrder()
{
    cout<<"[非递归]前序遍历：";
    
    if(root_ == nullptr)	
    {	return;	}
    
    stack<Node*> s;
    s.push(root_);
    while(!s.empty())
    {
        //取出栈顶元素并打印
        Node *top = s.top();
        s.pop();
        cout<<top->data_<<" ";
        
        //右孩子入栈
        if(top->right_ != nullptr)	
        {
            s.push(top->right_);
        }
        //左孩子入栈
        if(top->left_ != nullptr)
        {
           	s.push(top->left_);
        }
    }
    
    cout<<endl;
}
```

**中序遍历**

- 左中右   L  V  R

- ==一直有左孩子的话，左孩子一直入栈==，直到没有左孩子了，栈顶出栈打印（V)，然后看打印的结点有没有右孩子，有的话右孩子入栈，该右孩子又以LVR的方式打印，即有左孩子的话一直入栈......

  ![image-20210320152029327](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A0%E6%A0%91.img/image-20210320152029327.png)

```cpp
vector<int> inorderTraversal(TreeNode* root) 
{
    //左边有孩子就一直入栈
    if(root == nullptr) return vector<int>();

    vector<int> vec;
    stack<TreeNode*> sta;

    while(root != nullptr || !sta.empty())
    {
        while(root != nullptr)
        {
            sta.push(root);
            root = root->left;
        }
        //左边到底了，出栈打印一个
        TreeNode* cur = sta.top();	sta.pop();
        vec.push_back(cur->val);

        root = cur->right;
    }
    return vec;
}
```

**施磊代码**

```cpp
//非递归中序遍历
void n_inOrder()
{
    cout<<"[非递归]中序遍历：";
    if(root_ == nullptr)	
    {	return;	}
    
    stack<Node*> s;
    
    Node * cur = root_;
    
    while(!s.empty() || cur != nullptr)
    {
        //一直有左孩子的话，左孩子一直入栈
        if(cur != nullptr)
        {
            s.push(cur);
            cur = cur->left_;
        }
        else
        {
            //栈顶出栈打印（V)
            Node* top = s.top();
            s.pop();
            cout<<top->data_<<" ";

            //然后看打印的结点有没有右孩子，有的话右孩子入栈
            //该右孩子又以LVR的方式打印，即有左孩子的话一直入栈
            cur = top->right_;
        }
	}
    
    cout<<endl;
}
```

**后序遍历**

- 左右根  L  R  V
- ==求出 VRL，然后倒过来就是LRV，求VRL类似于前序遍历==
- 用前序遍历的类似方法求VRL，然后将输出的结果放入栈2，栈2的顺序输出就是LRV

```cpp
//非递归后序遍历
void n_postOrder()
{
    cout<<"[非递归]后序遍历：";
    if(root_ == nullptr)
    {
        return;
    }
    
    stack<Node*> s1;
    stack<Node*> s2;
    s1.push(root_);
    while(!s1.empty())
    {
        Node* top = s1.top();
        s1.pop();
        
        s2.push(top);  //V
        
        if(top->left_ != nullptr)
        {
            s1.push(top->left_);  //L，后访问的先入
        }
        if(top->right_ != nullptr)
        {
            s1.push(top->right_);  //R
        }   
    }
    
    //顺序输出s2
    while(!s2.empty())
    {
        cout<<s2.top()->data_<<" ";
        s2.pop();
    }
    
    cout<<endl;
}
```

> [二叉树的前序遍历](#二叉树的前序遍历)
>
> [二叉树的中序遍历](#二叉树的中序遍历)
>
> [二叉树的后序遍历](#二叉树的后序遍历)
>
> [二叉搜索树与双向循环链表](#二叉搜索树与双向循环链表)

# 求二叉树层数、深度

#### 递归解法

- 二叉树的深度等于左子树的深度和右子树的深度的最大值加1

```cpp
//递归求二叉树层数、深度，以下代码写在BSTree类内
int high(Node* node)
{
    if (node == nullptr)
    {
        return 0;
    }
    int left = high(node->left_);
    int right = high(node->right_);
    return left > right ?  left + 1 : right + 1;
}
int high()
{
    return high(root_);
}
```

#### 非递归解法（***）

**我的思路：**

- 利用层序遍历，但是每次出队列都要把一层的处理完

- > 相似思路的题目：[判断二叉树是否为满二叉树](#判断二叉树是否为满二叉树)

```cpp
//获取深度（非递归）
int GetDepth(BTNode * ptr)
{
	if (ptr == nullptr) return 0;
	int h = 0;
	queue<BTNode*> que;
	que.push(ptr);

	while (!que.empty())
	{
		int n = que.size(); //这一层的结点的个数
		while (n--)//要把这一层的出完
		{
			ptr = que.front();	que.pop();
			if (ptr->left != nullptr)		que.push(ptr->left);
			if (ptr->right != nullptr)		que.push(ptr->right);
		}
		h++;
	}
	return h;
}
```

> 相关题目
>
> [二叉树的最大深度](#二叉树的最大深度)

# 求二叉树结点个数

#### 递归解法

- 结点总数：左子树结点数+右子树结点数+1

```cpp
//求二叉树结点个数，左孩子结点数+右孩子结点数+1，以下代码写在BSTree类内
int high(Node* node)
{
    if (node == nullptr)
    {
        return 0;
    }
    int left = number(node->left_);
    int right = number(node->right_);
    return left + right + 1;
}
int number()
{
    return number(root_);
}
```

# BST树层序遍历

#### 递归解法

- 有多少层，就进行多少次深度递归遍历，每次遍历都会经过相应的层，到相应的层，打印该结点的值
- <img align='left' src="img/01BST%E6%A0%91.img/image-20210319134107684.png" alt="image-20210319134107684" style="zoom:50%;" />

```cpp
//层序遍历（递归），以下代码写在BSTree类内
void levelOrder(Node* node, int i)
{
    if (node == nullptr)
    {
        return;
    }
    if (i == 0)  //已经到了该打印的层
    {
        cout << node->data_ << " ";
        return;
    }
    //深度遍历二叉树上的每一个结点，每次往下，记录的层数减一，减到0就是该打印了
    levelOrder(node->left_, i - 1);
    levelOrder(node->right_, i -1);
}
void levelOrder()
{
    cout << "[递归]层序遍历: ";
    int h = high();//树的层数
    
    for (int i = 0; i < h; ++i)  //递归调用树的层数次，每到相应的层，就进行打印
    {
        levelOrder(root_, i);
    }
    cout << endl;
}
```

#### 非递归解法（***）

- 广度优先遍历借助队列，（上面的前中后序非递归遍历是深度优先，使用的是栈）

- ==根节点入队，队列不为空，出队一个打印，将该结点左右孩子入队==，然后再出队打印，左右孩子入队，再出队打印，左右孩子入队......

  ![image-20210320160405206](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A0%E6%A0%91.img/image-20210320160405206.png)

```cpp
//非递归层序遍历
void n_levelOrder()
{
    cout << "[非递归]层序遍历：";
    if (root_ == nullptr)
    {
        return;
    }

    queue<Node*> que;
    que.push(root_);

    while (!que.empty())
    {
        //队列不为空，出队一个并打印
        Node* cur = que.front();
        que.pop();
        cout << cur->data_ << " ";  
		
        //左右孩子入队
        if (cur->left_ != nullptr)
        {
            que.push(cur->left_);
        }
        if (cur->right_ != nullptr)
        {
            que.push(cur->right_);
        }
    }
    cout << endl;
}
```

> [从上到下打印二叉树](#从上到下打印二叉树)
>
> [从上到下打印二叉树II](#从上到下打印二叉树II)
>
> [从上到下打印二叉树III](#从上到下打印二叉树III)
>
> [二叉树的层序遍历](#二叉树的层序遍历)

# BST树区间元素查找

给定一棵二叉搜索树，找到元素值在[first, last]之间的结点

**思路：**

- BST树的==中序遍历是有序的递增序列==，利用中序遍历，遍历到结点值在[i, j]区间的，记录到容器

- 如果当前结点<=i，那么没必要在其左子树中继续查找；如果当前结点>=j，没必要在其右子树中继续查找

  ![image-20210321114450039](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210321114450039.png)

```cpp
//找到BST树中结点值在[i, j]区间的，并将值放入vec中
void findValues(vector<T> &vec, int i, int j)
{
    findValues(root_, vec, i, j);
}

void findValues(Node* node, vector<T> &vec, int i, int j)
{
    if(node != nullptr)
    {
        //L：在当前结点的左子树中搜索，如果当前结点<=i，那么没必要在其左子树中继续查找
        if(node->data_ > i)
        {
            findValues(node->left_, vec, i, j);
        }
        
        
        if(node->data_ >= i && node->data_ <= j)//V，满足区间，放入vec
        {
            vec.push_back(node->data_);
		}
        
        //R：在当前结点的右子树中搜索，如果当前结点>=j，没必要在其右子树中继续查找
        if(node->data_ < j)
        {
             findValues(node->right_, vec, i, j);
        }        
	}
}
```

# 判断一课二叉树是否是BST树

**思路1（递归）：**

- 利用BST树中序遍历是递增有序序列的性质，==中序遍历二叉树，每次比较该结点值是否大于前驱结点值==，大于的话，更新前驱，继续遍历，否则返回假

![image-20210321140111344](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210321140111344.png)

```cpp
bool isBSTree(Node* node, Node* &pre)//pre记录前一个结点
{
    if(node == nullptr)
    {
        return true;
    }
    if(!isBSTree(node->left_, pre));//L
    {
        return false;
    }
    
    if(pre != nullptr)//V
    {
        if(node->data_ < pre->data_)//递归结束条件：当前结点值比它前一个小，非递增序列，非BST
        {
            return false;
        }
    }
    pre = node;  //更新前驱
    
    return isBSTree(node->right_, pre);//R
}
bool isBSTree()
{
    Node* pre = nullptr;
    return isBSTree(root_, pre);
}
```

**思路2（非递归）：**

![image-20210403163421746](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210403163421746.png)

# 判断BST树子结构问题

给定一个二叉树，与一个子树，判断该子树是不是给出的二叉树的子结构，子结构可以看做二叉树的一部分

> 区分子树与子结构：
>
> 子树：若B是A的子树，则A包含B的所有结点，**并且B的叶子节点就是A的叶子节点**。也就是A只要包含了B的一个结点，就得包含这个结点下的所有节点。
>
> 子结构：若B是A的子结构，则A包含B的所有结点，**但B的叶子节点不一定是A的叶子节点**。也就是子结构B是A树的任意一部分。

**思路：**

- ==先在大树里面找到子结构的根节点，然后同时遍历子树与大树，进行比对==
- ![image-20210321174924873](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210321174924873.png)

```cpp
//判断子结构问题，该函数写在BSTree类内，可以直接访问root_
bool isChildTree(BSTree<T, Comp> &child)
{  
    if(child.root_ == nullptr)
    {
        return true;
    }   
    
    //先在大树里面找到子树的根节点
    Node* cur = root_;
    while(cur != nullptr)
    {
        if(cur->data_ == child.root_->data_)
        {
            break;
		}
        else if(comp_(cur->data_, child.root_->data_))
        {
            cur = cur->right_;
        }
        else
        {
            cur = cur->left_;
		}
    }
    if(cur == nullptr)//没有找到子树根
    {
        return false;
    }
    
    //找到了子树根在大树中的位置，同时遍历两个树进行比对
    return isChildTree(cur, child.root_);
}

bool isChildTree(Node* father, Node* child)
{
    if(father == nullptr && child == nullptr)
    {
        return true;
    }
    if(father == nullptr) //子树里有的结点，大树里没有
    {
        return false;
	}
    if(child == nullptr) //子树没有的结点，大树有
    {
        return true;
	}
    //判断当前结点值
    if(father->data_ != child->data_)
    {
        return false;
    }
    //同时判断左右孩子
    return isChildTree(father->left_, child->left_) 
        && isChildTree(father->right_, child->right_); 
}
```

# 二叉树的子结构

```cpp
输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)
B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:
给定的树 A:

     3
    / \
   4   5
  / \
 1   2
给定的树 B：

   4 
  /
 1
返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。
```

**我的思路：**

- ==先从A树中找到B树的根结点存起来，然后同时遍历根结点、B树来进行比对==

- 注意：在A树中找到的结点有可能有多个，每个都要与B树进行比对

  <img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210322162528879.png" alt="image-20210322162528879" style="zoom:50%;" />

```cpp
class Solution {
    //先从A树中找到B树的根结点, 然后同时遍历pub和B来进行比对
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) //A是大树，B是小树
    {
        if(B == nullptr)
        {
            return false;
        }

        //先从A树中找到B树的根结点，注意如果有重复的元素都要接收到
        vector<TreeNode*> vec = findRoot(A, B);

        //然后挨个用容器中的节点和B树来进行比对
        for(int i = 0; i < vec.size(); ++i)
        {
            if(isChild(vec[i], B))
            {
                return true;
            }
        }
        return false;    
    }

    //检查看pub和B是否相同
    bool isChild(TreeNode* pub, TreeNode * B)
    {
        if(pub == nullptr && B == nullptr)
        {
            return true;
        }
        if(pub == nullptr && B != nullptr)
        {
            return false;
        }
        if(pub != nullptr && B == nullptr)
        {
            return true;
        }
        if(pub->val != B->val)
        {
            return false;
        }

        return isChild(pub->left, B->left)
            && isChild(pub->right, B->right);
    }

    //在A树中找到B树的根结点并返回，有重复的都要返回
    vector<TreeNode*> findRoot(TreeNode * A, TreeNode * B)
    {
        vector<TreeNode*> vec; 
        if(B == nullptr)
        {
            return vec;
        }
        //存放找到的结点
        
        //层序遍历A来找，使用辅助队列
        queue<TreeNode*> que;
        que.push(A);

        while(!que.empty())
        {
            TreeNode* cur = que.front();
            que.pop();
            if(cur->val == B->val)
            {
                vec.push_back(cur);
            }

            if(cur->left != nullptr)
            {
                que.push(cur->left);
            }
            if(cur->right != nullptr)
            {
                que.push(cur->right);
            }
        }
        return vec;
    }
};
```

# BST树最近公共祖先LCA

LCA：Least Common Ancestors

**我的思路：**

- 因为二叉搜索树左小右大，所以可以遍历该树，如果pq比结点值都大，则在右子树中继续

- pq比结点值都小，在左子树中继续，如果==pq一个比该节点大，一个比结点小，那么这就是分叉点==，就是公共祖先

   <img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210321200712933.png" alt="image-20210321200712933" style="zoom:50%;" />

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) 
    {
        //因为二叉搜索树左小右大，所以可以遍历该树，如果pq比结点值都大，则在右子树中继续
        //比结点值都小，在左子树中继续，如果pq一个比该节点大，一个比结点小，那么这就是分叉点，就是公共祖先
        while(root != nullptr)
        {
            if(p->val > root->val && q->val > root->val)
            {
                root = root->right;
            }
            else if(p->val < root->val && q->val < root->val)
            {
                root = root->left;
            }
            else
            {
                return root;
            }
        }
        return root;
    }
};
```

> [二叉树的最近公共祖先](#二叉树的最近公共祖先)
>
> [二叉树中和为某一值的路径](#二叉树中和为某一值的路径)

# 二叉树的镜像翻转

```cpp
请完成一个函数，输入一个二叉树，该函数输出它的镜像。

     4
   /   \
  2     7
 / \   / \
1   3 6   9
镜像输出：
     4
   /   \
  7     2
 / \   / \
9   6 3   1
    
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
```

**我的思路：**

- 递归解法：前序遍历，==遍历到一个结点就交换其左右孩子，然后继续遍历左子树，右子树==

- 非递归解法：非递归解法：利用栈，交换当前结点左右孩子，将结点的左孩子与右孩子分别入栈，再出栈一个交换

```cpp
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) 
    {
        /*递归解法：
        if(root == nullptr) return root;
        
        TreeNode* tmp = root->left;
        root->left = root->right;
        root->right = tmp;
        
        mirrorTree(root->left);
        mirrorTree(root->right);
        return root;
        */

        //非递归解法：利用栈，交换当前结点左右孩子，将结点的左孩子与右孩子分别入栈，再出栈一个交换
        if(root == nullptr) return root;
        stack<TreeNode*> st;
        st.push(root);
        
        while(!st.empty())
        {
            TreeNode* node = st.top();
            st.pop();

            TreeNode* tmp = node->left;
            node->left = node->right;
            node->right = tmp;

            if(node->left != nullptr) 
            {
                st.push(node->left);
            }
            if(node->right != nullptr)
            {
                st.push(node->right);
            }
            
        }
        return root;
    }
};
```

# 二叉树的镜像对称

```cpp
给定一棵二叉树，判断其是否是镜像对称的二叉树
镜像对称的二叉树有如下特点：
     4
   /   \
  2     2
 / \   / \
1   3 3   1
```

**思路：**

- 传入根节点的左右结点，检查是否相同，然后再检查：

- 左边的结点的右孩子和右边结点的左孩子要相同  ==左右=右左？==

- 左边的结点的左孩子和右边结点的右孩子要相同  ==左左=右右？==

  <img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210322112108383.png" alt="image-20210322112108383" style="zoom:50%;" />

```cpp
//镜像对称
bool mirror2()
{
    if(root_ == nullptr)	return true;
    return mirror2(root_->left_, root_->right_);
}

bool mirror2(Node* node1, Node* node2)
{
    if(node1 == nullptr && node2 == nullptr)
    {
		return true;
    }
    if(node1 == nullptr || node2 == nullptr)  //两个里面有一个为空了
    {
        return false; 
	}
    //检查两个结点是否相同
    if(node1->data_ != node2->data_) 
    {
        return false;
	}
    //检查 左右==右左  左左==右右
    return mirror2(node1->right_, node2->left_)
        && mirror2(node1->left_, node2->right_);    
}
```

# 已知前序、中序，重建二叉树

**思路：**

- ==通过前序可以知道根是谁，通过中序可以知道根的左右两边有谁，进而知道左子树的前中序与右子树的前中序==，建立左右子树的时候，再根据该前序中序去重建

  ![image-20210322130545212](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210322130545212.png)


```cpp
//根据前序与中序，重建二叉树，ij是前序的起始末尾下标，mn是中序的起始末尾下标
void rebuild(int pre[], int i, int j, int in[], int m, int n)
{
    root_ = _rebuild(pre, i, j, in, m, n);
}
//根据前序与中序，重建二叉树，ij是前序的起始末尾下标，mn是中序的起始末尾下标，返回的是根节点
Node* _rebuild(int pre[], int i, int j, int in[], int m, int n)
{
    if(i > j || m > n)//前序/中序数组已经遍历完毕
    {
        return nullptr;
    }
    //拿前序的第一个节点创建根节点
    Node *node = new Node(pre[i]);
    
    //在中序数组里找到该根，进而知道左孩子与右孩子的中序与前序，建立左右子树的时候，再根据该前序中序去创建
    int k;
    for(k = m; k < n; ++k)
    {
        if(pre[i] == in[k])
        {
            break;
		}    
    }
    
    node->left_ = _rebuild(pre, i+1, i+(k-m), in, m, k-1);
    node->right_ = _rebuild(pre, i+(k-m)+1, j, in, k+1, n);
    
    return node;
}
```

# 已知中序、后序，重建二叉树

**我的思路：**

- ==通过后序可以知道根是谁，通过中序可以知道根的左右两边有谁，进而知道左子树的中后序与右子树的中后序==，建立左右子树的时候，再根据该中后序去重建
- ![image-20210404173429070](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210404173429070.png)

```cpp
//利用中序和后序重建二叉树，返回根结点
TreeNode* rebuild(vector<int> & inorder, vector<int> & postorder)
{
    TreeNode* root = rebuild(inorder, 0, inorder.size() - 1, 
                             postorder, 0, postorder.size()-1);
    return root;
}
TreeNode* rebuild(vector<int> & inorder, int i, int j, 
                  vector<int> & postorder, int m, int n)
{
    if(i > j || m > n)  //中序、后序已经遍历完毕
    {
        return nullptr;
    }
    //拿后序的第一个结点创建根节点
    TreeNode* node = new TreeNode(postorder[n]);

    //在中序序列中找到根节点，进而知道左孩子与右孩子的中序，建立左右子树的时候，再根据中序和后序去建立
    int k;
    for(k = i; k < j; ++k)
    {
        if(inorder[k] == postorder[n])
        {
            break;
        }
    }
    node->left = rebuild(inorder, i, k -1, postorder, m, m+k-i-1);
    node->right = rebuild(inorder, k+1, j, postorder, m+k-i, n-1);
    return node;
}
```



# 判断二叉树是否是平衡树

平衡树：任意结点的左右子树的高度<=1

**思路1：**

- 深度遍历二叉树到底，==回溯的时候判断每个结点的左右子树高度是否<=1==

```cpp
//调用之前实现的high方法，效率不好
bool isBalance(Node* node)
{
    if(node == nullptr)
    {
        return true;
	}
    if(!isBalance(node->left_))
    {
        return false;
	}
    if(!isBalance(node->right_))
    {
        return false;
    }
    
    int left = high(node->left_);//效率不好
    int right = high(node->right_);
    return abs(left-right) <= 1;
}
```

**思路2：**

- 遍历二叉树，给每个结点对应的函数栈上记录一个该结点所在的高度，==回溯的时候将高度返回回去==，用flag记录是否失衡

```cpp
bool isBalance()
{
    int h = 0;
    bool flag = true;  //true没失衡，false失衡
    isBalance2(root_, h, flag);
    return flag;
}

//递归的过程中记录了结点的高度值，
int isBalance2(Node* node, int h, bool & flag) //h：当前结点高度level
{
    if(node == nullptr)
    {
        return h;
	}
    int lh = isBalance2(node->left_, h+1, flag); //左子树高度
    if(!flag)	return lh;
    
    int rh = isBalance2(node->right_, h+1, flag);//右子树高度
    if(!flag)	return rh;
    
    if(abs(hl - rh)>1)
    {
        flag = false;
	}
    return max(lh, rh);
}
```

# 求BST树中序遍历倒数第k个结点

**思路：**

- 中序LVR，求倒数的可以==把中序LVR倒过来，求RVL的第k个结点==

```cpp
//求中序倒数第k个结点
int getValue(int k)
{
    Node* node = getVal(root_, k);
    if(node == nullptr)
        throw "no No.k";
    else
        return node->data_;
}
int i = 1; //记录到没到k
Node*getVal(Node* node, int  k)
{
    if(node == nullptr)
        return nullptr;
    
    Node* right = getVal(node->right_, k); //R
    if(right != nullptr)
    {
        return right;
    }
    
    if(i++ == k)//V
    {
   		return node;
    }
    
    return getVal(node->left_, k);  //L
}
```

# ------------------------------------

# 附录：相关的leetcode题

# 二叉搜索树中的插入操作

```cpp
给定二叉搜索树（BST）的根节点和要插入树中的值，将值插入二叉搜索树。 返回插入后二叉搜索树的根节点。 输入数据 保证 ，新值和原始二叉搜索树中的任意节点值都不同。
    
输入：root = [4,2,7,1,3], val = 5
输出：[4,2,7,1,3,5]
```

 <img src="img/01BST%E6%A0%91.img/image-20210318232739435.png" alt="image-20210318232739435" style="zoom:33%;" />

**我的思路：**

- 从根节点开始比较，找到val应该插入的位置，进行插入：生成节点，接到父结点指针域（每次记录父结点）
- 注意插入空树，要特殊处理

```cpp
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) 
    {
        //插入空树
        if(root == nullptr)
        {
            root = new TreeNode(val);
            return root;
        }
		
        //找到val应该插入的位置
        TreeNode * cur = root;
        TreeNode * parent = nullptr;
        while(cur != nullptr)
        {
            if(val > cur->val)
            {
                parent = cur;
                cur = cur->right;
            }
            else if(val < cur->val)
            {
                parent = cur;
                cur = cur->left;
            }
            else
            {
                break;
            }
        }
        //cur就是要插入的位置，父结点为parent，接到父结点指针域
        TreeNode* node = new TreeNode(val);
        if(val > parent->val)
        {
            parent->right = node;
        }
        else
        {
            parent->left = node;
        }
        return root;
    }
};
```

# 二叉树的最大深度

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

说明: 叶子节点是指没有子节点的节点。

    示例：
    给定二叉树 [3,9,20,null,null,15,7]
        3
       / \
      9  20
        /  \
       15   7
    返回它的最大深度 3 。
**我的思路：**

- 二叉树的深度等于左子树的深度和右子树的深度的最大值加1

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) 
    {
        if(root == nullptr) 
        {
            return 0;
        }
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```

# 删除二叉搜索树中的结点

```cpp
给定一个二叉搜索树的根节点 root 和一个值 key，删除二叉搜索树中的 key 对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。

一般来说，删除节点可分为两个步骤：

首先找到需要删除的节点；如果找到了，删除它。
说明： 要求算法时间复杂度为 O(h)，h 为树的高度。

root = [5,3,6,2,4,null,7]
key = 3
    5
   / \
  3   6
 / \   \
2   4   7

给定需要删除的节点值是 3，所以我们首先找到 3 这个节点，然后删除它。
一个正确的答案是 [5,4,6,2,null,null,7], 如下图所示。

    5
   / \
  4   6
 /     \
2       7

另一个正确答案是 [5,2,6,null,4,null,7]。

    5
   / \
  2   6
   \   \
    4   7
```

**我的思路：**

- 情况1：删除的结点没有孩子，将其父结点地址域置为nullptr
- 情况2：删除的结点有一个孩子，写入父结点地址域
- 情况3：删除的结点有两个孩子，找其前驱，把前驱写入要删除的结点，再删除前驱

```cpp
class Solution {
public:
    TreeNode* deleteNode(TreeNode* root, int key) 
    {
        //情况1：删除的结点没有孩子，将其父结点地址域置为nullptr
        //情况2：删除的结点有一个孩子，写入父结点地址域
        //情况3：删除的结点有两个孩子，找其前驱，把前驱写入要删除的结点，再删除前驱

        //先找到key，同时要保存key的父结点
        TreeNode * cur = root;
        TreeNode * par = nullptr;
        while(cur != nullptr)
        {
            if(key > cur->val)
            {
                par = cur;
                cur = cur->right;
            }
            else if(key < cur->val)
            {
                par = cur;
                cur = cur->left;
            }
            else
            {
                break;
            }
        } //出循环后，cur就是要删除的结点，par是其父结点

        //没找到要删除的结点
        if(cur == nullptr)
        {
            return root;
        }

        //情况3：删除的结点有两个孩子，找其前驱，把前驱写入要删除的结点，再删除前驱
        TreeNode* pre = nullptr;
        if(cur->left != nullptr && cur->right != nullptr)
        {
            par = cur;
            pre = cur->left;
            while(pre->right != nullptr)
            {
                par = pre;
                pre = pre->right;
            }
            //把前驱写入要删除的结点
            cur->val = pre->val; 
            cur = pre;//一会要删除pre，这里赋值方便统一处理
        }

        //情况1、2：将cur的孩子写入其父结点地址域
        TreeNode *child = cur->left;
        if(cur->left == nullptr)
        {
            child = cur->right;
        }
        if(par == nullptr)//删除的是根节点，根节点par结点为空，所以要特殊处理
        {
            root = child;
        }
        else
        {
            if(cur->val > par->val)
            {
                par->right = child;
            }
            else
            {
                par->left = child;
            }
        }
      
        //删除要删除的结点
        delete cur;
        return root;
    }
};
```

# 二叉搜索树中的搜索

```cpp
给定二叉搜索树（BST）的根节点和一个值。 你需要在BST中找到节点值等于给定值的节点。 返回以该节点为根的子树。 如果节点不存在，则返回 NULL。
给定二叉搜索树:

        4
       / \
      2   7
     / \
    1   3

和值: 2
你应该返回如下子树:
      2     
     / \   
    1   3
在上述示例中，如果要找的值是 5，但因为没有节点值为 5，我们应该返回 NULL。
```

**我的思路：**

- 从root开始，val比当前节点大，就在右边继续找，val比当前结点小，就在左边继续找

```cpp
//非递归
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) 
    {
        //从root开始，val比当前节点大，就在右边继续找，val比当前结点小，就在左边继续找
        TreeNode *cur = root;
        while(cur != nullptr)
        {
            if(val > cur->val)
            {
                cur = cur->right;
            }
            else if(val < cur->val)
            {
                cur = cur->left;
            }
            else
            {
                break;
            }
        }
        return cur;
    }
};

//递归
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) 
    {
        //从root开始，val比当前节点大，就在右边继续找，val比当前结点小，就在左边继续找
        while(root != nullptr)
        {
            if(val > root->val)
            {
                return searchBST(root->right, val);
            }
            else if(val < root->val)
            {
                return searchBST(root->left, val);
            }
            else
            {
                return root;
            }
        }
        return root;
    }
};
```

# 从上到下打印二叉树

```cpp
从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。
给定二叉树: [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
返回：[3,9,20,15,7]
```

**我的思路：**

- 层序遍历：先求出二叉树的层数，然后深度遍历二叉树，到某一层就输出某一层的结点

```cpp
class Solution {
private:
    vector<int> vec;
public:
    //二叉树的层数：左子树与右子树的层数最大值+1
    int high(TreeNode * root)
    {
        if(root == nullptr)
            return 0;
        return max(high(root->left)+1, high(root->right)+1);
    }

    //深度遍历二叉树，到某层就把某一层结点输出
    void levelOrder(TreeNode *root, int i)
    {
        if(root == nullptr)
        {
            return;
        }
        if(i == 0)
        {
            vec.push_back(root->val);
            return;
        }
        levelOrder(root->left, i -1);
        levelOrder(root->right, i - 1);
    }

    //先求出二叉树的层数，然后深度遍历二叉树，到某一层就输出某一层的结点
    vector<int> levelOrder(TreeNode* root) 
    {
        if(root == nullptr) return vec;

        int h = high(root);
        for(int i = 0; i< h; ++i)
        {
            levelOrder(root, i);
        }
        return vec;
    }
};
```

# 从上到下打印二叉树II

```cpp
从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。
例如:
给定二叉树: [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
返回其层次遍历结果：
[
  [3],
  [9,20],
  [15,7]
]
```

**我的思路1：**

- 层序遍历：先求出二叉树的层数，然后深度遍历二叉树，到某一层就输出某一层的结点到第层数个vector

```cpp
class Solution {
private:
    vector<vector<int>> vec;
public:
    //二叉树的层数：左子树与右子树的层数最大值+1
    int high(TreeNode * root)
    {
        if(root == nullptr)
            return 0;
        return max(high(root->left)+1, high(root->right)+1);
    }

    //深度遍历二叉树，到某层就把某一层结点输出
    void levelOrder(TreeNode *root, int i, vector<int>& xvec)
    {
        if(root == nullptr)
        {
            return;
        }
        if(i == 0)
        {
            xvec.push_back(root->val);
            return;
        }
        levelOrder(root->left, i -1, xvec);
        levelOrder(root->right, i - 1, xvec);
    }

    //先求出二叉树的层数，然后深度遍历二叉树，到某一层就输出某一层的结点
    vector<vector<int>> levelOrder(TreeNode* root) 
    {
        if(root == nullptr) return vec;

        int h = high(root);
        vec.resize(h);  //resize开辟空间同时构建对象，因为下面要用vec[i]

        for(int i = 0; i< h; ++i)
        {
            levelOrder(root, i, vec[i]);
        }
        return vec;
    }
};
```

**我的思路2：**

- 双队列：队列a存储第一层，队列b存储第二层

```cpp
class Solution {
public:
    vector<vector<int> > levelOrder(TreeNode* root) 
    {
        vector<vector<int>> vvec;
        if(root == nullptr)    return vvec;
        
        queue<TreeNode*> quea;
        queue<TreeNode*> queb;
        quea.push(root);
        TreeNode* cur;
        
        while(!quea.empty() || !queb.empty())
        {
            vector<int> veca;
            vector<int> vecb;
            while(!quea.empty())
            {
                cur = quea.front(); quea.pop();
                veca.push_back(cur->val);
                if(cur->left != nullptr)
                {
                    queb.push(cur->left);
                }
                if(cur->right != nullptr)
                {
                    queb.push(cur->right);
                }
            }
            if(!veca.empty())    vvec.push_back(veca);
            while(!queb.empty())
            {
                cur = queb.front(); queb.pop();
                 vecb.push_back(cur->val);
                if(cur->left != nullptr)
                {
                    quea.push(cur->left);
                }
                if(cur->right != nullptr)
                {
                    quea.push(cur->right);
                }   
            }
            if(!vecb.empty())    vvec.push_back(vecb);
        }
        return vvec;
    }
};
```

# 从上到下打印二叉树III(Z字形)

```cpp
请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。
给定二叉树: [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
返回其层次遍历结果：
[
  [3],
  [20,9],
  [15,7]
]
```

**我的思路1：**

- 先求出二叉树的层数，然后深度遍历二叉树，到某一层就把这层的结点存入对应的数组
- 最后将需要逆置的数组进行逆置

```cpp
class Solution {
private:
    vector<vector<int>> vec;
public:
    int high(TreeNode* root) //求二叉树的深度：左子树与右子树深度的最大值再加1
    {
        if(root == nullptr) return 0;
        return max(high(root->left), high(root->right)) + 1;
    }

    void levelOrder(TreeNode* root, int i, vector<int> & xvec)
    {
        if(root == nullptr)
        {
            return;
        }
        if(i == 0)
        {
            xvec.push_back(root->val);
        }
        levelOrder(root->left, i-1, xvec);
        levelOrder(root->right, i - 1, xvec);
    }

    vector<vector<int>> levelOrder(TreeNode* root) 
    {
        //先求出二叉树的层数，然后深度遍历二叉树，到某一层就把这层的结点存入对应的数组，
        if(root == nullptr) return vec;
        int h = high(root);
        //初始化vec（构建对象）
        vec.resize(h);

        for(int i = 0; i < h; ++i)
        {
            levelOrder(root, i, vec[i]);
        }

        //将需要逆置的数组内部进行逆置
        for(int i = 0 ; i < h; ++i)
        {
            if(i % 2 != 0)
            {
                //逆置
                int start = 0;
                int end = vec[i].size() - 1;
                while(start < end)
                {
                    int tmp = vec[i][start];
                    vec[i][start] = vec[i][end];
                    vec[i][end] = tmp;
                    start++;
                    end--;
                }
            }
        }
        return vec;
    }
};
```

**我的思路2：**

- 准备两个栈，根结点放入栈1，栈1不空时，出元素，将元素的==左右==孩子放入栈2

- 栈2不为空时，出元素，将元素的==右左==孩子放入第一个栈

  ![image-20210330221223640](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210330221223640.png)

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) 
    {
        //准备两个栈，根结点放入栈1，栈1不空时，出元素，将元素的左右孩子放入栈2
        //栈2不为空时，出元素，将元素的右左孩子放入第一个栈
        vector<vector<int>> vvec;
        if(root == nullptr) return  vvec;

        stack<TreeNode*> st1;
        stack<TreeNode*> st2;

        st1.push(root);
        while(!st1.empty() || !st2.empty()) 
        {    
            if(!st1.empty())
            {
                vector<int> vec1; //存当前层的结点
                while(!st1.empty())
                {
                    TreeNode* cur = st1.top(); st1.pop();
                    vec1.push_back(cur->val);

                    if(cur->left != nullptr)    st2.push(cur->left);
                    if(cur->right != nullptr)   st2.push(cur->right);
                }
                vvec.push_back(vec1);
            }
            
            if(!st2.empty())
            {
                vector<int> vec2;
                while(!st2.empty())
                {
                    TreeNode* cur = st2.top(); st2.pop();
                    vec2.push_back(cur->val);

                    if(cur->right != nullptr)   st1.push(cur->right);
                    if(cur->left != nullptr)    st1.push(cur->left);
                }
                vvec.push_back(vec2);
            }
            
        }
        return vvec;
    }
};
```



# 二叉树的前序遍历

给你二叉树的根节点 `root` ，返回它节点值的 **前序** 遍历。

**我的思路1：**

- 递归：根，前序遍历左子树，前序遍历右子树

```cpp
class Solution {
public:
    void preorderTraversal(TreeNode* root, vector<int> & vec)
    {
        if(root == nullptr)
        {
            return;
        }
        vec.push_back(root->val);
        preorderTraversal(root->left, vec);
        preorderTraversal(root->right, vec);
        return;

    }
    vector<int> preorderTraversal(TreeNode* root) 
    {
        //前序遍历：根，前序遍历左子树，前序遍历右子树
        vector<int> vec;
        preorderTraversal(root, vec);
        return vec;
    }
};
```

**我的思路2：**

- 非递归：前序遍历：根左右，根节点入栈，栈不为空，出栈输出，然后右节点入栈，左节点入栈

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) 
    {
        //前序遍历：根左右
        //根节点入栈，栈不为空，出栈输出，然后右节点入栈，左节点入栈
        if(root == nullptr)
        {
            return vector<int>();
        }
        
        vector<int> vec;

        stack<TreeNode*> st;
        st.push(root);

        while(!st.empty())
        {
            TreeNode* cur = st.top();
            st.pop();
            vec.push_back(cur->val);
            
            if(cur->right != nullptr)
            {
                st.push(cur->right);
            }
            if(cur->left != nullptr)
            {
                st.push(cur->left);
            }
        }
        return vec;
    }
};
```

# 二叉树的中序遍历

给你二叉树的根节点 `root` ，返回它节点值的 **中序** 遍历。

**我的思路1：**

- 递归：中序遍历左子树，根，中序遍历右子树

```cpp
class Solution {
public:
    void inorderTraversal(TreeNode* root, vector<int> & vec)
    {
        if(root == nullptr) 
        {
            return;
        }
        inorderTraversal(root->left, vec);
        vec.push_back(root->val);
        inorderTraversal(root->right, vec);
    }
    vector<int> inorderTraversal(TreeNode* root) 
    {
        if(root == nullptr)
        {
            return vector<int>();
        }

        vector<int> vec;
        inorderTraversal(root, vec);
        return vec;
    }
};
```

**我的思路2：**

- 非递归：左边的有就一直入栈，左边到底了，输出栈顶，看其右孩子有没有，有的话就入栈继续

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) 
    {
        //左边的有就一直入栈，到空的时候输出栈顶，看该节点右边有没有，有的话就入栈，继续
        if(root == nullptr)
        {
            return vector<int>();
        }
        vector<int> vec;
        stack<TreeNode*> st;

        while(root != nullptr || !st.empty())
        {
            if(root != nullptr)//左边有就一直入栈
            {
                st.push(root);
                root = root->left;
            }
            else//左边到底了，输出栈顶，看右边的有没有，有的话就入栈继续
            {
                TreeNode* top = st.top();
                st.pop();
                vec.push_back(top->val);

                root = top->right;
            }
        }
        return vec;
    }
};
```

# 二叉树的后序遍历

给定一个二叉树，返回它的 **后序** 遍历。

**我的思路1：**

- 递归：后序左子树，后序右子树，根

```cpp
class Solution {
public:
    void postorderTraversal(TreeNode* root, vector<int> & vec)
    {
        if(root == nullptr)
        {
            return;
        }
        postorderTraversal(root->left, vec);
        postorderTraversal(root->right, vec);
        vec.push_back(root->val);
        return;
    }

    vector<int> postorderTraversal(TreeNode* root) 
    {
        if(root == nullptr)
        {
            return vector<int>();
        }

        vector<int> vec;
        postorderTraversal(root, vec);

        return vec;
    }
};
```

**我的思路2：**

- 非递归：左右根，先求出根右左的序列，再逆序输出
- 求根右左，类似于前序遍历，根节点入栈，出栈并输出，左孩子入栈，右孩子入栈，栈顶出栈，继续左入，右入
- 最后把输出结果导出到vector中

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) 
    {
        //非递归：左右根，先求出根右左的序列，再逆序输出
        if(root == nullptr)
        {
            return vector<int>();
        }

        //求根右左，类似于前序遍历，根节点入栈，出栈并输出，左孩子入栈，右孩子入栈，栈顶出栈，继续左入，右入
        vector<int> vec;
        stack<TreeNode*> sta;  
        stack<int> rst;//存输出结果的栈

        sta.push(root);
        while(!sta.empty())
        {
            TreeNode* top = sta.top();
            sta.pop();
            rst.push(top->val);

            if(top->left != nullptr)
            {
                sta.push(top->left);
            }
            if(top->right != nullptr)
            {
                sta.push(top->right);
            }
        }

        while(!rst.empty())
        {
            vec.push_back(rst.top());
            rst.pop();
        }
        return vec;
        
    }
};
```

# 二叉树的层序遍历

```cpp
从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。
给定二叉树: [3,9,20,null,null,15,7],

    3
   / \
  9  20
    /  \
   15   7
返回：
[3,9,20,15,7]
```

**我的思路：**

- 递归：[从上到下打印二叉树](#从上到下打印二叉树)
- 非递归：广度优先遍历，用队列，根节点入队列，出队列，左孩子入队，右孩子入队，出队，左入，右入，出

```cpp
//非递归代码
class Solution {
private:
    vector<int> vec;
public:
    //广度优先遍历，用队列，根节点入队列，出队列，左孩子入队，右孩子入队，出对列，左入，右入，出
    vector<int> levelOrder(TreeNode* root) 
    {
        if(root == nullptr)
        {
            return vector<int>();
        }
        queue<TreeNode*> que;
        que.push(root);

        vector<int> vec;

        while(!que.empty())
        {
            TreeNode* front = que.front();
            que.pop();
            vec.push_back(front->val);

            if(front->left != nullptr)
            {
                que.push(front->left);
            }
            if(front->right != nullptr)
            {
                que.push(front->right);
            }
        }
        return vec;
    }
};
```

# 二叉搜索树与双向循环链表

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中的第一个节点的指针。

<img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210323104558329.png" alt="image-20210323104558329" style="zoom:50%;" />

**我的思路1：**

- 中序遍历，记录前驱结点，每次将当前结点的左孩子指向前驱，前驱的右孩子指向当前结点
- 注意处理pre为空时，以及最后要首尾相连

```cpp
Node* treeToDoublyList(Node* root) 
{
    //中序遍历，记录前驱结点，每次将当前结点的左孩子指向前驱，前驱的右孩子指向当前结点
    //注意处理pre为空时，以及最后要首尾相连
    if(root == nullptr) return nullptr;

    Node* newroot = nullptr;
    Node* pre = nullptr;

    stack<Node*> sta;

    while(root != nullptr || !sta.empty())
    {
        while(root != nullptr)
        {
            sta.push(root);
            root = root->left;
        }

        root = sta.top();  sta.pop();

        if(pre == nullptr)
        {
            newroot = root;  //root是中序遍历第一个结点
        }
        else
        {
            pre->right = root;
            root->left = pre;
        }
        pre = root;
        root = root->right;
    }

    //首尾相连
    newroot->left = pre;  
    pre->right = newroot; 

    return newroot;
}
```

**我的思路2：**

- 中序遍历二叉搜索树的到有序序列，将结果存到vector中，然后修改左右孩子指针分别指向前驱和后继
- 注意先处理0个结点和1个结点的特殊情况

```cpp
class Solution {
public:
    Node* treeToDoublyList(Node* root) 
    {
        //先处理0个结点和1个结点的特殊情况
        if(root == nullptr)//0个
        {
            return root;
        }
        if(root->left == nullptr && root->right == nullptr)//1个结点
        {
            root->left = root;
            root->right = root;
            return root;
        }

        //中序遍历该二叉搜索树，将结果存到vector中，然后修改指针指向
        vector<Node*> vec = inOrder(root);    
        int size = vec.size();

        for(int i = 0; i < size; ++i)
        {
            if(i == 0) //第一个节点的前驱是最后一个节点
            {
                vec[i]->left = vec[size - 1];
            }
            else
            {
                vec[i]->left = vec[i-1];
            }
            
            if(i == size - 1)//最后一个节点的后继是第一个节点
            {
                vec[i]->right = vec[0];
            }
            else
            {
                vec[i]->right = vec[i+1];
            }
            
        }
        return vec[0];
    }

    //中序遍历
    vector<Node*> inOrder(Node* root)
    {
        vector<Node*> vec;
        if(root == nullptr)
        {
            return vec;
        }

        Node* cur = root;
        stack<Node*> st;

        while(!st.empty() || cur != nullptr)
        {
            //左边能找到就一直往左下找        
            if(cur != nullptr)
            {
                st.push(cur);
                cur = cur->left;
            }
            else
            {
                //输出最左下的结点
                Node* top = st.top();
                vec.push_back(top);
                st.pop();
                cur = top->right;
            }       
        }
        return vec;
    }
};
```

# 判断二叉树是否为满二叉树

**我的思路1：**

- 层序遍历二叉树，统计所有结点个数，判断最后结点个数是否是2^(h+1) - 1，（根的h是0）

**我的思路2：**

- 利用层序遍历的思路，每次都处理一整层并且判断该层结点个数是否为2^h个

```cpp
bool IsFull(BTNode * ptr)
{
	if (ptr == nullptr) return true;
	int h = 0;
	queue<BTNode*> que;
	que.push(ptr);

	while (!que.empty())
	{
		int n = que.size(); //这一层的结点的个数
        if(n != pow(2, h))
        {
            return false;
		}
		while (n--)//要把这一层的出完
		{
			ptr = que.front();	que.pop();
			if (ptr->left != nullptr)		que.push(ptr->left);
			if (ptr->right != nullptr)		que.push(ptr->right);
		}
		h++;
	}
	return true;
}
```

**或者这样：**不用pow(2^h)计算该层结点个数，用he += he，1  2  4  8  16......

<img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210329224603614.png" alt="image-20210329224603614" style="zoom:50%;" />

# 判断二叉树是否为完全二叉树

**我的思路：**

- 利用层序遍历的思想，把结点入队列（空指针也入队列），遍历到空指针，然后判断队列里面的是不是都是空指针，都是空的话说明是完全二叉树，有非空的结点说明不是完全二叉树

   <img align='left' src="img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210329231934173.png" alt="image-20210329231934173" style="zoom:50%;" />

```cpp
bool Is_comp(BTNode* ptr)
{
    bool res = true;
    if(ptr == nullptr)	return res;
    queue<BTNode*> qu;
    qu.push(ptr);
    
    while(!qu.empty())//所有结点入队列，直到结点为空
    {
        ptr = qu.front();	qu.pop();
        if(ptr == nullptr)	break;
        qu.push(ptr->left);
        qu.push(ptr->right);
	}
    while(!qu.empty())//判断队列里面的是不是都是空指针
    {
        ptr = qu.front();	qu.pop();
        if(ptr != nullptr)
        {
            res = false;
            break;
        }
    }
    return res;
}
```

# 二叉树中查找指定值的元素

**我的思路：**

- 判断根节点是不是val，不是的话，在左子树中找，左子树中没找到在右子树中找

```cpp
//二叉树中查找val
BTNode* FindValue(BTNode* ptr, ElemType val)
{
	if (ptr == nullptr || ptr->data == val)
	{
		return ptr;
	}
	else
	{
		BTNode* p = FindValue(ptr->left, val);
		if (p == nullptr)
		{
			p = FindValue(ptr->right, val);
		}
		return p;
	}
}
```

# 二叉树中查找给定结点的双亲

**我的思路：**

- 判断要查找的结点是不是根节点，是的话返回空，（根节点没有双亲）
- 否则判断根节点的左孩子右孩子是否等于给定的节点，不等的话在左子树中查找，左子树中没找到在右子树中找

```cpp
//寻找child结点的双亲
BTNode* FindParent(BTNode* ptr, BTNode* child)
{
	if (ptr == nullptr || child == nullptr || ptr == child)//如果是根节点，双亲是空
	{
		return nullptr;
	}
	else
	{
		if (ptr->left == child || ptr->right == child)
		{
			return ptr;
		}
		else
		{
			BTNode* cur = FindParent(ptr->left, child);
			if (cur == nullptr)
			{
				cur = FindParent(ptr->right, child);
			}
			return cur;
		}
	}
}
```

# 二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

 <img src="../../../%25E5%25AD%25A6%25E4%25B9%25A0/%25E6%259C%25AC%25E5%259C%25B0md%25E6%2596%2587%25E4%25BB%25B6/%25E7%25AE%2597%25E6%25B3%2595/img/3.11%25E5%25BC%2580%25E5%25A7%258B%25E7%259A%2584%25E9%25A2%2598.img/image-20210320230339195.png" alt="image-20210320230339195" style="zoom:50%;" />

```cpp
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出：3
解释：节点 5 和节点 1 的最近公共祖先是节点 3 。
```

**我的思路：**

- 深度优先遍历，在二叉树中找到两个节点的路径，分别存入两个数组，然后找两个数组的分叉点

- 二叉树中找节点的路径：从顶向下找，记录路径，到底部的时候，弹出一个，继续找，找到的话修改标记，没必要继续遍历

> 这道题找路径的方法类似于这道题：[二叉树中和为某一值的路径](#二叉树中和为某一值的路径)

```cpp
class Solution {
private:
    vector<TreeNode*> path_p;
    vector<TreeNode*> path_q;
    bool flag = false;
public:
    //深度优先遍历(类似前序，但每次要更新路径)在二叉树中找到结点key的路径，并返回路径到path
    void findPath(TreeNode* root, TreeNode* key, vector<TreeNode*>& path)
    {
        if (root == nullptr || key == nullptr || flag)
        {
            return;
        }

        path.push_back(root);
        if (root == key)
        {
            flag = true;// 已经找到
            return;
        }
        if (root->left == nullptr && root->right == nullptr)  //遍历到底了，弹出一个，回溯
        {
            path.pop_back();
            return;
        }
        findPath(root->left, key, path);
        findPath(root->right, key, path);
        if (!path.empty() && !flag)//每次遍历完一个结点的左右子树，如果没有成功找到，回溯的时候要从路径中剔除该结点
        {
            path.pop_back();
        }
        return;
    }

    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q)
    {
        //深度优先遍历，在二叉树中找到两个节点的路径，分别存入两个数组，然后找两个数组的分叉点
        if (root == nullptr || p == nullptr || q == nullptr)
        {
            return nullptr;
        }

        findPath(root, p, path_p);
        flag = false;
        findPath(root, q, path_q);

        int i = 0;
        for (i = 0; i < path_p.size() && i < path_q.size(); ++i)
        {
            if (path_p[i] != path_q[i])
            {
                break;
            }
        }
        return path_p[i-1];      
    }
};
```

# 二叉树中和为某一值的路径

```cpp
输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。
给定如下二叉树，以及目标和 target = 22，

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
返回:
[
   [5,4,11,2],
   [5,8,4,5]
]

```

**我的思路：**

- 深度优先遍历二叉树，每次遍历的时候记录之前的结点，遍历到叶子结点的时候看之前的和是否为target
- 是的话将该路径保存，不是的话该路径删除一个元素，回溯

```cpp
class Solution {
private:
    vector<int> onepath; //子路径
    vector<vector<int>> sumpath; //所有路径
public:
    vector<vector<int>>& pathSum(TreeNode* root, int target)//返回引用可以省大量时间与空间
    {
        //深度优先遍历二叉树，每次遍历的时候记录之前的结点，遍历到叶子的时候看之前的和是否为target
        //是的话将该路径保存，不是的话该路径删除一个元素，回溯
        if(root == nullptr) return sumpath;
        
        onepath.push_back(root->val);

        if (root->left == nullptr && root->right == nullptr) //遍历到底的时候看之前的和是否为target
        {
            int sum = 0;
            for (auto it : onepath)
            {
                sum += it;
            }
            if (target == sum)
            {
                sumpath.push_back(onepath);
            }  
            if (!onepath.empty())
            {
                onepath.pop_back();
            } 
            return sumpath;
        }
        
        pathSum(root->left, target);
        pathSum(root->right, target);

        if (!onepath.empty())
        {
            onepath.pop_back();
        } 
    
        return sumpath;
    }
};
```

# 判断是否为BST树的后序遍历序列

```cpp
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。

参考以下这颗二叉搜索树：

     5
    / \
   2   6
  / \
 1   3

输入: [1,6,3,2,5]
输出: false
```

**我的思路：**

- 传入的后序遍历序列，排序后得到的有序数组就是中序遍历序列
- 利用中序、后序重建二叉树，然后判断重建出来的二叉树是不是BST树
- 中序、后序重建二叉树辅助图示：

![image-20210404173159911](img/01%EF%BC%9ABST%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.img/image-20210404173159911.png)

```cpp
class Solution {
public:
    //传入的后序遍历序列，排序后得到的有序数组就是中序遍历序列
    //利用中序、后序重建二叉树，然后判断重建出来的二叉树是不是BST树
    bool verifyPostorder(vector<int>& postorder) 
    {
        //排序得到中序序列
        vector<int> inorder = postorder;
        sort(inorder.begin(), inorder.end());

        //利用中序和后序重建二叉树
        TreeNode* root = rebuild(inorder, postorder);

        //判断该二叉树是否为BST树
        return isBST(root);
    }

    //树的结点
    struct TreeNode
    {
        TreeNode(int val): data(val), left(nullptr), right(nullptr) {}
        int data;
        TreeNode* left;
        TreeNode* right;
    };

    //利用中序和后序重建二叉树，返回根结点
    TreeNode* rebuild(vector<int> & inorder, vector<int> & postorder)
    {
        TreeNode* root = rebuild(inorder, 0, inorder.size() - 1, 
                                 postorder, 0, postorder.size()-1);
        return root;
    }
    TreeNode* rebuild(vector<int> & inorder, int i, int j, 
                      vector<int> & postorder, int m, int n)
    {
        if(i > j || m > n)  //中序、后序已经遍历完毕
        {
            return nullptr;
        }
        //拿后序的第一个结点创建根节点
        TreeNode* node = new TreeNode(postorder[n]);

        //在中序序列中找到根节点，进而知道左孩子与右孩子的中序，建立左右子树的时候，再根据中序和后序去建立
        int k;
        for(k = i; k < j; ++k)
        {
            if(inorder[k] == postorder[n])
            {
                break;
            }
        }
        node->left = rebuild(inorder, i, k -1, postorder, m, m+k-i-1);
        node->right = rebuild(inorder, k+1, j, postorder, m+k-i, n-1);
        return node;
    }

    //判断root是否为BST树
    bool isBST(TreeNode* root)
    {
        //中序遍历，看是否是递增的
        if(root == nullptr)
        {
            return true;
        }
        TreeNode* pre = nullptr;
        stack<TreeNode*> st;

        while(!st.empty() || root != nullptr)
        {
            while(root != nullptr)
            {
                st.push(root);
                root = root->left;
            }
            root = st.top();    st.pop();
            if(pre != nullptr && pre->data >= root->data)
            {
                return false;
            }
            pre = root;
            root = root->right;
        }
        return true;
    }
};
```

**出错的思路：**

- 输入的是后序遍历序列，用这个数组构建出一棵BST树
- 然后后序遍历新BST树，得到的新后序序列和传进来后序序列比对
- 思路1出错原因：同一个序列插入顺序不同，构建出来的BST树是不同的，后序序列可能确实是BST树的后序序列，但是却并不是用后序序列去插入构建成的

<img align='left' src="../../../%25E5%25AD%25A6%25E4%25B9%25A0/%25E6%259C%25AC%25E5%259C%25B0md%25E6%2596%2587%25E4%25BB%25B6/%25E7%25AE%2597%25E6%25B3%2595/img/3.11%25E5%25BC%2580%25E5%25A7%258B%25E7%259A%2584%25E9%25A2%2598.img/image-20210404154359476.png" alt="image-20210404154359476" style="zoom:50%;" />

```cpp
class Solution {
public:
    //思路1出错原因：同一个序列插入顺序不同，构建出来的BST树是不同的

    //输入的是后序遍历序列，用这个数组构建出一棵BST树
    //然后后序遍历新BST树，得到的新后序序列和传进来的比对
    bool verifyPostorder(vector<int>& postorder) 
    {
        //用传进来的后序序列构建新BST树
        TreeNode* root = nullptr;
        for(auto & it : postorder)
        {
            insert(root, it);
        }

        //后序遍历新BST树
        vector<int> newpost = postOrder(root);

        //新后序序列和传进来的比对
        for(int i = 0; i < postorder.size(); ++i)
        {
            if(postorder[i] != newpost[i])
            {
                return false;
            }
        }
        return true;
    }

    //树的结点
    struct TreeNode  
    {
        TreeNode(int val):data(val), left(nullptr), right(nullptr)  {}
        int data;
        TreeNode* left;
        TreeNode* right;
    };

    //BST树的插入
    void insert(TreeNode* &root, int val)
    {
        if(root == nullptr)
        {
            root = new TreeNode(val);
            return;
        }

        TreeNode* cur = root;
        TreeNode* par = nullptr;
        while(cur != nullptr && cur->data != val)
        {
            par = cur;
            cur = val < cur->data ? cur->left : cur->right;
        }
        if(val < par->data)
        {
            par->left = new TreeNode(val);
        }
        else
        {
            par->right = new TreeNode(val);
        }
        return;
    }

    //后序遍历：左右根->根右左->再倒过来
    vector<int> postOrder(TreeNode* root)
    {
        vector<int> vec;
        stack<TreeNode*> st;

        if(root == nullptr)
        {
            return vec;
        }

        st.push(root);

        //根右左
        while(!st.empty())
        {
            TreeNode* cur = st.top(); st.pop();
            vec.push_back(cur->data);

            if(cur->left != nullptr) //后访问的先入
            {
                st.push(cur->left);
            }
            if(cur->right != nullptr)
            {
                st.push(cur->right);
            }
        }

        //倒过来
        vector<int> rt;
        for(auto it = vec.rbegin(); it != vec.rend(); ++it)
        {
            rt.push_back(*it);
        }
        return rt;
    }
};
```

