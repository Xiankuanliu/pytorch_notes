# д��ǰ��
�ڶ�Tensorflow�ĺ��Դ������˲�⣨�μ�[tensorflowԴ�����ϵ����������](https://www.cnblogs.com/jicanghai/p/9589412.html)��֮�󣬺�����������ѧϰ��ܵ�ʵ�ֽ��жԱȣ����ݿ�ܵ����г̶ȣ���ѡ����Pytorch��Pytorch�ĺ�˺�����ֱ�Ӹ�����Caffe2����˱������Caffe2Դ���coreģ������˼򵥲�⡣

# Ŀ¼
- ���ݴ洢���ʾ
    - storage
    - tensor
    - blob
    - qtensor
- ����
    - observer observable
    - operator
    - ������
    - operator_schema
    - context
- ����ͼ
    - graph
    - net
    - transform
- ����ʱ
    - allocator
    - db
    - registry
    - module
    - scope_guard
    - workspace
    - init

# 1. ���ݴ洢���ʾ
## 1.1 storage
Caffe2�ж����ݴ洢����ײ��������Storage����ʵ������ָ��StorageImpl�Ĺ���ָ�룬���߰����������͡�����ָ�롢���������������豸����Ϣ��Storage�Ķ������£�
```
using Storage = std::shared_ptr<StorageImpl>;
class StorageImpl {
  public:
    //...
  protected:
    using DataPtr = std::shared_ptr<void>;
    int64_t capacity_ = 0;
    DataType data_type_;
    DataPtr data_ptr_;
    DeviceType device_type_ = CPU;
};
```
## 1.2 tensor
Caffe2�е�����ͳһʹ��Tensor��ʾ��Tensor��TensorImplʵ�֣����߰���һ��Storage��
```
graph LR
    Tensor-->|����|TensorImpl
    TensorImpl-->|����|Storage
    Storage-->|ָ��|StorageImpl
```

TensorImpl�Ķ������£�
```
class TensorImpl {
  public:
    //...
  protected:
    using DimVector = std::vector<TIndex>;
    DimVector dims_; //������ά��
    TIndex size_ = -1; //�����а�����Ԫ������
    Storage storage_; //�ײ�洢
};
```
Tensor���Ǽ̳���TensorImpl���������ڲ�������һ��ָ��TensorImpl��ָ�룬���£�
```
class Tensor final {
  protected:
    using TensorImplPtr = c10::intrusive_ptr<TensorImpl, UndefinedTensorImpl>;
    TensorImplPtr impl_;
  //...
};
```
��Tensor�ķ������ã�ͨ���ض����TensorImplʵ�֡�

## 1.3 blob
Blob��һ��������������һ��ָ������ָ��ָ���ڴ���������ͣ���Caffe2�У��󲿷������Blob������һ��ָ��Tensor��ָ�롣
```
class Blob {
  public:
    //...
  private:
    TypeMeta meta_;
    void* pointer_ = nullptr;
    DestroyCall destroy_ = nullptr;
};
```

Ϊ�˷����Blob���д��䣬�����������л��ͷ����л����࣬�ֱ���BlobSerializerBase��BlobDeserializerBase���Լ���Ӧ��ΪTensor׼�������л��ͷ����л��ࡣ
```
graph TB
    BlobSerializerBase-->|����|TensorSerializer
    BlobDeserializerBase-->|����|TensorDeserializer
```

## 1.4 qtensor
�;��ȵ�������Ϊ�˱��ڿ��ٽ��е;��ȵ������˷����㡣����������ǣ��ø��͵�λ������ʾ���������磬��3��bit��ʾ�޷�����������4��bit��ʾ�з����������;���������������΢��ʧģ�;��ȵ�����£���󽵵ͼ��㸴�ӶȺʹ洢�ռ��С��

# ����
## 2.1 Observer Observable
Caffe2ʹ��ObserverBase��Observable������ʵ���˹۲���ģʽ��ObserverBase�ǻ����۲������û�����ͨ���̳д��ഴ���µĹ۲�������Observable�ǿɱ��۲����ԣ��û�����ͨ���̳д����ÿɹ۲����ԡ�

ObserverBase�ṩ�˹۲�����ͳһ�ӿڣ��Ƚϼ򵥣�
```
class ObserverBase {
  public:
    virtual void Start() {}
    virtual void Stop() {}
    T* subject() const {
        return subject_;
    }
  protected:
    T* subject_;
};
```
���У�subject_��ʾ���۲�����ָ�롣

Observable��װ�˿ɱ��۲����ԣ��ڲ�������һ���۲������б��ṹ���£�
```
class Observable {
  public:
    using Observer = ObserverBase<T>;
    const Observer* AttachObserver(std::unique_ptr<Observer> observer){} //��ӹ۲���
    std::unique_ptr<Observer> DetachObserver(const Observer* observer_ptr){} //����۲���
    virtual size_t NumObservers() {
        return num_observers_;
    } //�۲���������
    void StartAllObservers(){} //�������й۲���
    void StopAllObservers(){} //�ر����й۲���
  private:
    Observer* observer_cache_;
    size_t num_observers_ = 0;
  protected:
    std::vector<std::unique_ptr<Observer>> observer_list_; //�۲����б�
};
```
## 2.2 Operator
Operator��������ľ���ʵ�֣��൱��Tensorflow�е�kernel��Operator�̳���OperatorBase�������߼̳���Observable��������Caffe2�У�����������������һ���ɹ۲�Ķ���
```
graph LR
    Observable-->|����|OperatorBase
    OperatorBase-->|����|Operator
```
OperatorBase������˲�����Ҫ�Ļ�������Ԫ�غͽӿڣ�
```
class OperatorBase {
  private:
    Workspace* operator_ws_;
    std::shared_ptr<const OperatorDef> operator_def_;
    DeviceOption device_option_;
    std::string engine_;
    std::string type_;
    vector<const Blob*> inputs_;
    vector<Blob*> outputs_;
};
```
OperatorBase�а����������������ڴ�ָ�룬�ɼ�����Caffe2�У�Operator��������һ������ʱ�Ķ�������Tensorflow��Op��������ͬ����Tensorflow�У�Op��һ������ʱ���󣬽��涨�˲��������ͺ�Ŀ�꣬���������������ݣ�����ļ���ʵ������ͨ��Kernel��ɵġ�

Operator�̳���OperatorBase�ࣺ
```
class Operator : public OperatorBase {
  public:
    bool Run(int stream_id = 0) final {...}
    bool RunAsync(int stream_id = 0) final {...}
    virtual bool RunOnDevice() = 0;
};
```
ʵ���ϣ�Run��RunAsync���ն�������RunOnDevice�����ʵ�ʵļ��㡣

���������Ҫʹ��һЩc10�ж���Ĳ�������Ҫ����ת��Ϊ��Caffe2�п��Ե��õĲ���������ͨ�����µĺ����ת����
```
REGISTER_C10_OPERATOR_FOR_CAFFE2_DISPATCH(C10Add, C2MyAddOpName)
```
���������У����ǰ�һ��C10Add��������װ��C2MyAddOpName������������ʹ�á�Ϊ��ʵ��������ܣ�Caffe2���ṩ��һ����װ�࣬C10OperatorWrapper��

## 2.3 ������
Ϊ�˶Բ����󵼣�Caffe2�Ƴ���һ���������������࣬GradientMakerBase�������û��������ĳ�������ĵ���������������ݳ�Ա���£�
```
//Ϊ�ܼ���ϡ���blob�ṩͳһ�Ľӿ�
struct GradientWrapper {
    string dense_;
    string indices_;
    string values_;
    inline bool IsDense(){}
    inline bool IsSparse(){}
    inline bool IsEmpty(){}
};
class GradientMakerBase {
  protected:
    const OperatorDef& def_;
    const vector<GradientWrapper>& g_output_;
    vector<GradientWrapper> g_input_;
};
```
�ɼ���GradientMakerBase���ṩ������������Լ�ԭ�������û����Ը���ԭ���������Ƶ�����

## 2.3 operator_schema
OpSchema�ǶԲ����ľ�̬�������൱��Tensorflow�е�Op����������Ϣ���£�
```
class OpSchema {
  private:
    string type_;
    string file_;
    string doc_;
    string onnx_schema_;
    std::vector<Argument> args_{};
    std::vector<std::pair<const char*, const char*>> input_desc_{};
    std::vector<std::pair<const char*, const char*>> output_desc_{};
    int line_ = 0;
    int min_input_ = 0;
    int max_input_ = std::numeric_limits<int>::max();
    int min_output_ = 0;
    int max_output_ = std::numeric_limits<int>::max();
    bool private_ = false;
    bool inputs_can_cross_devices_ = false;
    std::function<bool(int)> num_inputs_allowed = [](int) { return true; }
    std::function<bool(int)> num_outputs_allowed = [](int) { return true; }
    std::function<bool(int,int)> num_inputs_outputs_allowed_ = [](int,int) { return true; }
    std::function<int(int)> calculate_output_;
    std::function<bool(int,int)> inplace_allowed_ = [](int,int){}
    std::function<bool(int,int)> inplace_enforced_ = [](int,int){}
    TensorInferenceFunctionType tensor_inference_function_ = {...}
    std::unique_ptr<CostInferenceFunctionType> cost_inference_function_ = nullptr;
    DeviceInferenceFunctionType device_inference_function_ = {...}
};
```
����Caffe2Ҳ�ṩ��һ������OpSchema��ע����OpSchemaRegistry�����£�
```
class OpSchemaRegistry {
  private:
    static CaffeMap<string, OpSchema>& map();
};
```

## 2.4 context
Caffe2�е�context����ʵ����Tensorflow�е�OpKernelContext��Ϊ������ʵ�ʼ����ṩͨ�õ�֧�֣���Ҫ�����ڴ濽���Ľӿڡ�����ʵ�ʵ�Context�����̳���BaseContext����Caffe2Ϊ����׼����һ����׼��Context�ӿڣ�CPUContext�ࡣ���⣬Ҳͬ��ΪGPU׼����һ��CUDAContext�ࡣ
```
graph LR
    BaseContext-->|����|CPUContext
    BaseContext-->|����|CUDAContext
```

# 3. ����ͼ
## 3.1 graph
Graph��ʾͼ�Ľṹ��ͼ�����ڵ㣬�ڵ����������
```
graph LR
    Graph-->|����|Node
    Node-->|����|OperatorDef
```
Node���������ݳ�Ա��
```
class Node {
  public:
    OperatorDef op;
    bool active = true; //�����Ƿ�transformationɾ��
    std::map<int, std::vector<string>> parents;
    std::vector<int, std::vector<string>> children;
}
```
Graph������˽�����ݳ�Ա��
```
class Graph {
  private:
    NetDef netdef_;
    std::set<string> external_input_;
    std::set<string> external_output_;
    std::vector<Node> nodes_;
}
```

## 3.2 net
Net��һ�������е�Graph��������һ��ͼ�����С����������Լ����ǵ������ġ����̳���Observable����������һ���ɹ۲�Ķ������ݳ�Ա���£�
```
class NetBase : public Observable<NetBase>{
  public:
    virtual bool Run(){...}
    virtual bool RunAsync();
  protected:
    vector<string> external_input_;
    vector<string> external_output_;
    string name_;
    vector<const Event*> events_;
    std::shared_ptr<const NetDef> net_def_;
};
```

NetBase���������������࣬��һ����AsyncNetBase�����������첽ִ����������������ݺͽӿڣ�
```
class AsyncNetBase : public NetBase {
  public:
    bool RunAsync() override;
  protected:
    bool canSchedule(...);
    std::vector<OperatorBase*> operators_;
    std::vector<dag_utils::OperatorNode> operator_nodes_;
    std::vector<std::vector<int>> chains_;
    std::vector<dag_utils::OpGraphNode> chain_nodes_;
    dag_utils::ExecutionChains execution_chains_;
};
```

�ڶ�����SimpleNet������ʾ��һ�ֶ�ͼ�ĵ��̵߳�˳��ִ��ģʽ��
��������DAGNetBase������ʾ��һ�ֶ�ͼ�Ķ��̵߳�dagִ��ģʽ��
��ص�net���γ���һ���̳���ϵ��
```
graph TB
    Observable-->|����|NetBase
    NetBase-->|����|AsyncNetBase
    AsyncNetBase-->|����|AsyncSchedulingNet
    NetBase-->|����|DAGNetBase
    DAGNetBase-->|����|DAGNet
    NetBase-->|����|SimpleNet
    DAGNetBase-->|����|AsyncDAGNet
    AsyncNetBase-->|����|AsyncPollingNet
```

## 3.3 transform
transform��һ�����Caffe2��NetDef�ṹ�Ĳ���������NetDef��Ϊ���룬����µľ����任��NetDef�����Ĺ������������
- �Ӿɵ�NetDef�й���һ��ͼ������ͼ�б����˽ڵ��������Ϣ��
- ��ͼ��ƥ��ָ����ģʽ���ҵ�����Ҫ���ĵ���ͼ��
- ���µĲ����滻ƥ�䵽����ͼ��
- ����ͼ����һ���µ�NetDef�����أ�

Transform���ܵ�ʵ�֣��������������ܺ��������£�
- PatternRule��ģʽ���򣩣��������˶���һ����ͼ��һ���ڵ㣬�Ƿ���Խ�����ڵ���������ͼ�У�
- ValidatorRule����֤���򣩣���������һ����ͼ�Ƿ���ƥ��ģ�
- ReplaceRule���滻���򣩣�����һ��ƥ�����ͼ�����滻��

���õ�ģʽ���£�
- CONNECTED_SUBGRAPH��������ͼ����ֻ��ƥ�����ӵ���ͼ���������ͼ(1)-->(2)-->(3)-->(4)�����ܹ�ƥ�䵽[2,3]��[4,3]��������ƥ�䵽[2,4]��
- SORTED_WRT_EXECUTION_ORDER��ִ����ģʽ����ֻ��ƥ�����ִ��˳�����ͼ���ڵ�֮�䲻һ����Ҫ�����ӣ�����GeneralģʽҪ�죬�������ͼ(1)-->(2)-->(3)-->(4)��������ƥ�䵽[2,4],[3,4]��������ƥ�䵽[3,1]��[4,3]��
- GENERAL��������ƥ�䵽�κ���ͼ�����磬����ͼ(1)-->(2)-->(3)-->(4)��˵��������ƥ�䵽��ͼ[2,4]��[3,4]��[4,2,1]�ȣ�

# 4. ����ʱ
## 4.1 allocator
�ڴ��������
```
graph TB
    CPUAllocator-->|����|DefaultCPUAllocator
    CPUAllocator-->|����|PinnedCPUAllocator
```

## 4.2 db
DB���Ƕ�kv�洢�ĳ��󡣰��������ڶ�ȡDB���ݵ�Cursor�࣬����дDB���ݵ�Transaction�࣬DB��ȡ�İ�����DBReader����DBReader�������л��ͷ����л���DBReaderSerializer��DBReaderDeserializer�ࡣ
```
graph TB
    DB-->|������ʱ���α���|Cursor
    DB-->|д����ʱ��������|Transaction
    DB-->|�����ݰ�װ|DBReader
    DBReader-->|���л�|DBReaderSerilizer
    DBReader-->|�����л�|DBReaderDeserilizer
```

## 4.3 registry
ע���࣬keyΪ�ַ�����value����Ϊ������ࡣ�ṹ���£�
```
class Registry {
  private:
    CaffeMap<SrcType, Creator> registry_;
    CaffeMap<SrcType, string> help_message_;
};
```

## 4.4 module
�鿴Caffe2�������ģ�飬�Լ�����ָ��ģ�顣ģ��ָ���Ƕ�̬���ӿ⡣

## 4.5 scope_guard
�ǡ���ʼ������Դ��ȡ��ԭ���ʵ�֣�����֤�ˣ��������ʽ˵����������ִ�оͻ��뿪��ǰ��scope��

## 4.6 workspace
Workspace���������е�����ʱ���󣬰���blob��net������������Щ�����ӵ���ߣ��������Щ������й���
```
class Workspace {
  private:
    typedef CaffeMap<string, unique_ptr<Blob>> BlobMap;
    BlobMap blob_map_;
    typedef CaffeMap<string, unique_ptr<NetBase>> NetMap;
    NetMap net_map_;
    const string root_folder_;
    const Workspace* shared_;
    std::unordered_map<string, std::pair<const Workspace*, string>> forwarded_blobs_;
    std::unique_ptr<ThreadPool> thread_pool_;
    std::mutex thread_pool_creation_mutex_;
    std::shared_ptr<Bookkeeper> bookkeeper_;
};
```

## 4.7 init
��ʼ������Caffe2�����л��������л����ǣ�����Ҫ�ڻ�����ʼ�������еĺ���ע�ᵽע�����У���ʼ��ʱ�����ڲ�ͬʱ�����в�ͬע�����еĺ��������ĵĺ������£�
```
CAFFE2_API bool GlobalInit(int* pargc, char*** argv);
```
������ʼ�����̷�Ϊ������
- ������ͨ��REGISTER_CAFFE2_EARLY_INIT_FUNCTIONע��ĺ�����
- �ٽ���Caffe�������в�������������־��¼ϵͳ��
- �������ͨ��REGISTER_CAFFE2_INIT_FUNCTIONע��ĺ�����