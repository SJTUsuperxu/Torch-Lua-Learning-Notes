# Torch/Lua Learning Notes
***
### 1. local 
**local** 关键字非常重要，在一个程序段中(测试/模块)， **变量建议都设为局部变量**(加上 **local** 关键字)，否则可能会出现很多莫名其妙的错误。
以下面这段程序为例，该程序是要读一段文本并匹配得出文本中有多少个"y"。
```
f= assert(io.open("test.txt"), "w")
data = f:read(5)
num = 0
print(data)
repeat
    for k in string.gmatch(data, ".") do
        if k == "y" then num = num+1 end 
    end
    data = f:read(5)
until not data
print(num)
```
上面的程序看似没有任何问题，但它会报错: **bad argument #1 to 'gmatch' (string expected, got table.** 这个错误是非常诡异的，首先，data在之前已经定义过，但**由于受到全局影响，data类型发生了改变**，即不是string类型（**read返回的就是string**）。现在在data前面预先定义local data，程序如下：
```
local data
f= assert(io.open("test.txt"), "w")
data = f:read(5)
local num = 0 
print(data)
repeat
    for k in string.gmatch(data, ".") do
        if k == "y" then num = num+1 end 
    end
    data = f:read(5)
until not data
print(num)
```
修正后，程序运行正常，问题解决。

***
### 2. **nn** package functions summary (**Torch**)
- **nn.SplitTable(dim, [nInputDims])(InputTensor)**
  - 输出沿着某维度dim对InputTensor分割所得的若干sub-tensor
  - nInputDims是可选参数，它可以指定对输入的tensor维度。
- **nn.CAddTable()/CMulTable()({tensor1, tensor2, ...})**
  - 输出tensor1，tensor2...的"和/积"，注意tensors大小都得相同
- **nn.Narrow(dim, start_index, size)(InputTensor)**
  - 输出InputTensor的第dim维的tensor，tensor的起始下标是start_index，长度为size。
  - e.g. 取一个二维tensor的第二行 <=> nn.Narrow(1, 2, 1)(tensor)
- **nn.Linear(InputDim, OutputDim)(InputTensor)**
  - nn中的全连接层,InputDim和OutputDim分别表示输入输出Tensor的维度。InputTensor只能是1-D vector 或者2-D matrix。一般情况下，InputTensor是1-D，返回的结果是1-D tensor，维度是Outputdim。
  - **但是**， 当**InputTensor是2-D**时，默认是将tensor中的**每行视为batch中的一个样本**，以batch的形式输入输出，记tensor size为**rows * InputDim**，则**输出的结果是2-D tensor**，size是**rows * OutputDim**


### 3. Torch functions
- **resize/view/reshape**
  - resize never copies memory. reshape always copies memory. view never copies memory.
  - resize literally resizes the tensor. if a tensor's underlying storage is too small, it calls realloc to expand the storage to an appropriate size: https://github.com/torch/torch7/blob/master/lib/TH/generic/THTensor.c#L687 How realloc does the mallocing (i.e. whether it expands the memory in-place or allocates completely new memory) is handled at the system level. You should not rely on resize keeping the same memory if you ever change nElement of the tensor
  - reshape creates a new tensor and always copies over the memory of src tensor over.
  - view never copies, ever. worst case it errors out. view does not have an exact implementation in TH, but view's closest equivalent is this constructor: https://github.com/torch/torch7/blob/master/lib/TH/generic/THTensor.h#L42

- **split(tensor, size, [dim])**
  - 输出大小为size的tensors，tensors是沿着维度dim对tensor分割得到的。dim默认是1。
  
***

### 4. Some tips in Torch
- data是一个N维tensor, 若要访问第k维的数据块，可以这样进行：
  - data[{{},...,{start_index, end_index}, ...,{}}]
  - start_index 和 end_index 分别表示第k维数据块的起止index
