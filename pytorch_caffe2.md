# д��ǰ��
��һƪ���¶�Caffe2�е�coreģ������˼򵥲��[Caffe2Դ�����֮core](https://www.cnblogs.com/jicanghai/p/9689726.html)����ͷһ������ģ��Ҳû���ٴ��룬����˳�ֶ�����һ�飬Ŀ���Ǵ����˽�ÿ��ģ������ݺ�Ŀ�꣬��һ�����Caffe2�������ܡ����ݲ��࣬�����������¡�

# Ŀ¼
- core
- proto
    - caffe2.proto
    - hsm.proto
    - metanet.proto
- cuda_rtc
- db
- distributed
- ideep
- image
- mkl
- mobile
- mpi
- observers
- onnx
- operators
- opt
- perfkernels
- predictor
- queue
- sgd
- transform
- util
- python
- contrib

# core
�μ�[Caffe2Դ�����֮core](https://www.cnblogs.com/jicanghai/p/9689726.html)

# proto
������Caffe2�г��õ�protobuf���壬�ǳ���Ҫ�����ǰ��������ļ����н���
## caffe2.proto
������TensorProto������ʾ�������л���Ľ����������������ά�ȡ��������͡�����ֵ�����ơ������豸����Ϣ�����£�
```
message TensorProto {
    repeated int64 dims = 1;
    optional DataType data_type = 2 [default = FLOAT];
    repeated float float_data = 3 [packed = true];
    repeated int32 int32_data = 4 [packed = true];
    optional bytes byte_data = 5;
    repeated bytes string_data = 6;
    repeated double double_data = 9 [packed = true];
    repeated int64 int64_data = 10 [packed = true];
    optional string name = 7;
    
    //�������л�֮ǰ���ڵ��豸
    optional DeviceOption device_detail = 8;
    //������chunks�е�λ��
    message Segment {
        required int64 begin = 1;
        required int64 end = 2;
    }
    optional Segment segment = 11;
}
```

��coreģ���н�����Caffe2Ϊ��֧�ֵ;��ȵ�ģ��ѵ���������qtensor����ʱû����ϸ�������ı��ʣ�ʵ����qtensor�Ƕ�ԭ���������˹�һ��������ȥbias�ٳ���scale��Ȼ��Խ�����е;��ȱ�ʾ����ʡ�洢�ռ䡣�����qtensor�����л�����У���Ҫ�Թ�һ���Ĳ������м�¼�����£�
```
message QTensorProto {
    repeated int64 dims = 1;
    required int32 precision = 2;
    required double scale = 3;
    required double bias = 4;
    required bool is_signed = 5;
    repeated int32 data = 6 [packed = true];
    optional string name = 7;
    optional TensorProto.DataType data_type = 8 [default = INT32];
}
```

���ڶ��Tensor������ʹ��TensorProto�ĸ����汾��TensorProtos���洢����Ȼ��ֻ��Խ�С����������������Ƚϴ󣬽���ʹ��DB�洢��

������������״��Ҳ��һ���ṹ����ʾ��TensorShape���ǵ���Tensorflow�У�����������״��ĳЩά�ȣ�������ǰ���ܲ�������ȫ֪�������������TensorShape�Ķ����У�����Ӳ�����δ֪����ά��������
```
message TensorShape {
    repeated int64 dims = 1;
    optional TensorProto.DataType data_type = 2 [default = FLOAT];
    repeated int32 unknown_dims = 3;
    optional bool unknown_shape = 4 [default = false];
    optional string name = 5;
}
```

�������ڶԲ�����������������ĵ�OperatorDef���壩��һ�������Ĳ���Ҫô���������ĸ����͡����ͻ����ַ������ݣ�Ҫô�����������͵����飬���£�
```
message Argument {
    optional string name = 1;
    optional float f = 2;
    optional int64 i = 3;
    optional bytes s = 4;
    optional NetDef n = 8;
    repeated float floats = 5;
    repeated int64 ints = 6;
    repeated bytes strings = 7;
    repeated NetDef nets = 9;
}
```

ĿǰCaffe2֧�ֵ��豸���ͣ�
```
enum DeviceType {
    CPU = 0;
    CUDA = 1;
    MKLDNN = 2;
    OPENGL = 3;
    OPENCL = 4;
    IDEEP = 5;
    HIP = 6;
    COMPILE_TIME_MAX_DEVICE_TYPES = 7;
    ONLY_FOR_TEST = 20901701;
}
```

ĿǰCaffe2���ڲ�ͬ�豸������proto������һ�µģ����ĳ���豸û�а������е�ĳ���ֶΣ���ô����ֶν������ԡ�
```
message DeviceOption {
    optional int32 device_type = 1 [default = 0]; //0 is CPU
    optional int32 cuda_gpu_id = 2;
    optional uint32 random_seed = 3;
    optional string node_name = 4;
    optional int32 numa_node_id = 5;
    repeated string extra_info = 6;
    optional int32 hip_gpu_id = 7;
}
```

�������ǲ����Ķ��壺
```
message OperatorDef {
    repeated string input = 1; //�����blob����
    repeated string output = 2; //�����blob����
    optional string name = 3;
    optional string type = 4; //���������ͣ��Ӳ���ע�����д�����������ʱ����Ҫ�����Ϣ
    optional string type = 4;
    repeated Argument arg = 5;
    optional DeviceOption device_option = 6; //������������Ҫ���豸
    
    //���ڵ�ǰ������˵���������ָ���������豸�ж���������棬�������ָ��һ�������ʵ�����档����û�ָ����һ�����棬�����������Caffe2�Ķ����ư��в������ڣ���ôʹ��Ĭ������
    optional string engine = 7;
    
    //�������룬��Tensorflow�еĿ����������ƣ�������е��Ⱥ�˳�򣬶��������ݵ����롣�����ڵ���ʱ��Net��ʹ��
    repeated string control_input = 8;
    
    //is_gradient_op����������״�ƶϣ�shape inference����Tensorflow�����ƣ�ʱʹ�ã�û������ʱ������
    optional bool is_gradient_op = 9 [default = false];
    optional string debug_info = 10;
}
```

������NetDef�Ķ��壺
```
message NetDef {
    optional string name = 1;
    repeated OperatorDef op = 2;
    optional string type = 3; //network��ִ�з�ʽ��Ĭ����simple
    optional DeviceOption device_option = 5; //����net�����в������豸��Ϣ�����������ÿ��Ա����ÿ��������������
    repeated Argument arg = 6; //����������num_workers������ͼ������ִ�е�ʱ��worker������
    
    repeated string external_input = 7;
    repeated string external_output = 8;
}
```

Caffe2��Ҳ������Tensorflow�������е������㣬��ʹ����һ���ṹ����ExecutionStep�����£�
```
message ExecutionStep {
    optional string name = 1;
    
    //ExecutionStepҪô���԰���һ��substep�ļ��ϣ�Ҫô���԰���һЩҪ���е�network�����ƣ������߲���ͬʱ������
    repeated ExecutionStep substep = 2;
    repeated string network = 3;
    
    //��ǰ�ĵ�����Ҫ���е��ִΣ�substeps��networks��Ҫ��˳��ִ�У�ÿ��ִ�б���Ϊһ�ֵ���
    optional int64 num_iter = 4;
    
    //����ִ�н������ж�����
    optional string criteria_network = 5;
    
    //�������ֶα����ã���ô�������Ե�ִ��
    optional int64 run_every_ms = 11;
    
    //����sub-steps����˳��ִ�л��ǲ���ִ��
    optional bool concurrent_substeps = 6;
    
    //һ�������жϵ�ǰִ���Ƿ���Ҫ�ս�ı�־
    optional string should_stop_blob = 9;
    
    //���Ϊ�棬��ǰִ�н�ִ��һ�Σ�ע�����should_stop_blob��Чʱ����Ч
    optional bool only_once = 10;
    
    //�Ƿ�Ϊ��ǰִ�й���һ����workspace
    optional bool create_workspace = 12;
    
    //��ִ�еĲ��ж�
    optional int32 num_concurrent_instances = 13;
}
```

���˵һ��ExecutionStep��һ�ε���ִ�У���ôPlan����һ��������ִ�мƻ������߰���ǰ�ߣ�
```
message PlanDef {
    optional string name = 1;
    repeated NetDef netowrk = 2;
    repeated ExecutionStep execution_step = 3;
}
```

������Щ�ڲ�������Tensor��Blob��Caffe2���������µĽṹ��
```
message BlobProto {
    optional string name = 1;
    optional string type = 2;
    optional TensorProto tensor = 3;
    optional bytes content = 4;
    optional QTensorProto qtensor = 5;
    optional int32 content_num_chunks = 6;
    optional int32 content_chunk_id = 7;
}
```

����Ƕ�DBReader�������л��Ķ���
```
message DBReaderProto {
    optional string name = 1;
    optional string source = 2;
    optional string db_type = 3;
    optional string key = 4;
}
```

## hsm.proto
Word2Vec������Google�����һ��ģ�ͣ�Ŀ���Ǹ������Ͽ��ô�Ƕ�루embedding��������Ϊ�����ѵ�����ٶ���������ּ�����һ���Ǹ�������Negative Sampling��������һ�־���Hierarchical Softmax����ˣ�Caffe2ר�������һ��HSM����������ļ�������ľ�����֮��ص�proto�����ǽ�����proto���ƣ�����Ƚ���Ȼ��
```
message NodeProto;
message TreeProto;
message HierarchyProto;
message PathProto;
message PathNodeProto;
```

## metanet.proto
MetaNetDef������˼�壬������NetDef��Ԫ���ݡ���ṹ���£�
```
message MetaNetDef {
    repeated BlobMap blobs = 1;
    repeated NetsMap nets = 2;
    optional ModelInfo modelInfo = 3;
    repeated PlanMap plans = 4;
    repeated StringMap applicationSpecificInfo = 5;
}
```
���У���Ӧ��xxMap�ṹ�ܼ򵥣����Ǽ�ֵ�ԣ�ModelInfo��ԱȽϸ��ӣ����ǿ�����ϸ�Ķ��壺
```
message ModelInfo {
    optional string project = 1;
    optional string modelClass = 2;
    optional string version = 3;
    optional string predictorTtype = 4;
    optional string modelId = 5;
}
```

# cuda_rtc
cuda��������صĸ������롣

# db
��Caffe2��ִ�й����У���Ҫ�ظ�ʹ�ú͹���Ĳ������ᱻ��¼��һ��db���С���coreģ�������ǽ��ܹ���db����һ��kv�洢�����������4��Caffe2�л��õ���db�����£�
```
graph TB
    db-->|����|LevelDB
    db-->|����|LMDB
    db-->|����|ProtoDB
    db-->|����|ZmqDB
```

# distributed
Caffe2�ķֲ�ʽʵ�֣������ⲿ�洢�����湲��Ĳ��������õ��ⲿ�洢�����ļ���redis��

�ⲿ�洢�ľ����StoreHandler����ʾ�������������µĺ���API��
```
class StoreHandler {
  public:
    virtual void set(...) = 0;
    virtual std::string get(...) = 0;
    virtual int64_t add(...) = 0;
    virtual bool check(...) = 0;
    virtual void wait(...) = 0;
};
```
��Ӧ������ͼ�У�����4����store������op��֮��Ӧ�����£�
```
graph TB
    Operator-->|����|StoreSetOp
    Operator-->|����|StoreGetOp
    Operator-->|����|StoreAddOp
    Operator-->|����|StoreWaitOp
```
�ղ��ᵽ�ˣ����õĴ洢��ʽΪ�ļ��洢��redis�洢����Ӧ�����ִ洢�����
```
graph TB
    StoreHandler-->|����|RedisStoreHandler
    StoreHandler-->|����|FileStoreHandler
```
���⣬�������������洢�Ĳ��������£�
```
graph TB
    Operator-->|����|FileStoreHandlerCreateOp
    Operator-->|����|RedisStoreHandler
```

# ideep
Ŀǰ����������庬�塣

# image
����ͼ��Ĳ�������������Ҫ���Ƕ���ͼ���ȡ�Ĳ�����ImageInputOp�����̳���PrefetchOperator��������ͼ���ȡ��һϵ�й��ܡ�

# mkl
MKLȫ����Intel Math Kernel Library����Ӣ�ض��ṩ����ѧ���Ŀ⣬���Դ�������ѧ���̽����˴�����������Ż������������MKL��صĲ������塣ע�⣬Tensorflow��Ҳ�õ���MKLȥ�Ż���ѧ���㣬ֻ����������ͼ�Ż��Ĺ����У���MKL��Ϊһ��ͼ�Ż����������룬��Caffe2�н�MKLֱ�����뵽�˲����ڲ���

# mobile
����ƶ�ƽ̨�����⴦�����廹û����

# mpi
Caffe2�еķֲ�ʽ���㣬ͨ��mpiʵ�֡�mpi�ĺ����������ڲ�ͬ�����ϵķֲ�ʽ�����У��������ݴ������Ϣͬ�������mpi�еĺ��Ĳ���������Broadcast��Reduce�ȣ�Caffe2�������˶�Ӧ�Ĳ�����ִ�У��������£�
```
graph TB
    Operator-->|����|MPICreateCommonWorldOp
    Operator-->|����|MPIBroadcastOp
    Operator-->|����|MPIReduceOp
    Operator-->|����|MPIAllgatherOp
    Operator-->|����|MPIAllreduceOp
    Operator-->|����|MPISendTensorOp
    Operator-->|����|MPIReceiveTensorOp
```

# observers
������4�ֲ�ͬ�۲����Ķ��壬���£�
- operator_attaching_net_observer�������net�е�ÿһ��operator��ӹ۲�����
- profile_observer�������ÿ������������ͼ��ִ�����Ľ��й۲죻
- runcnt_observer�������ÿ��������������ͼ�����д������й۲죻
- time_observer�������ÿ��������������ͼ������ʱ����й۲죻

# onnx
Ŀǰ���������

# operators
�����ľ��嶨���������������޴�û���ü�ϸ����

# opt
�Ż���ص���ͺ�������Tensorflowһ����Caffe2Ҳ��ͨ����ͼ�����ķ�ʽʵʩ�Ż������е��Ż����������̳���OptimizationPass�����ľ��嶨�����£�
```
class OptimizationPass {
  public:
    OptimizationPass(NNModule* nn) : nn_(nn) {}
    virtual void run() = 0;
    virtual ~OptimizationPass(){}
    
  protected:
    NNModule* nn_;
};
```

# perfkernels
�����Ż���ص�kernel��

# predictor
һ��predictor����һ��������ȷ�����˵�net�������ѧϰ�У�����ͨ����Ѵ�ѧϰ��ģ�ͱ�ʾΪnet��Ȼ��ͨ��������ͼ���㣬ȷ��ģ�Ͳ�������netת��Ϊpredictor���������ǿ���predictor�Ľṹ��
```
class Predictor {
  public:
    Predictor(const NetDef& init_net, const NetDef& run_net, Workspace* parent = nullptr, bool run_init = true, int optimization = 1);
    Predictor(PredictorConfig config);
    
    //�����Ƕ�()�����أ���������õ����
    bool operator()(const TensorMap& inputs, TensorList* outputs);
    bool operator()(const TensorMap& inputs, TensorList* outputs);
    bool operator()(const TensorMap& inputs, TensorMap* outputs);
    
    const NetDef& def() const {
        return *config_.predict_net;
    };
    
    Workspace* ws(){
        return config_.ws.get();
    };
    const std::vector<std::string>& input_names() const {
        return config_.input_names;
    }
    const std::vector<std::string>& output_names() const {
        return config_.output_names;
    }
  private:
    bool run_map_workspace(const TensorMap& inputs);
    PredictorConfig config_;
};
```
���У�Predictor������Ҫ��һ��˽�����ݳ�Ա��config_�����ǿ���PredictorConfig�Ķ��壺
```
struct PredictorConfig {
    std::shared_ptr<PredictorParameters> parameters;
    std::shared_ptr<NetDef> predict_net;
    std::vector<std::string> input_names;
    std::vector<std::string> output_names;
    std::vector<std::string> parameter_names;
    std::shared_ptr<Workspace> ws;
};
```

# queue
��������BlobsQueue�Ķ��壺
```
class BlobsQueue : public std::enable_shared_from_this<BlobsQueue> {
  public:
    bool blockingRead(...);
    bool blockingWrite(...);
    void close();
  private:
    size_t numBlobs_;
    std::mutex mutex_;
    std::condition_variable cv_;
    int64_t reader_{0};
    int64_t writer_{0};
    std::vector<std::vector<Blob*>> queue_; //���Ķ�������
    const std::string name_;
};
```
ע�⿴���е�����Ԫ��queue_��������BlobsQueue�ĺ��Ķ������ݡ�

���⣬BlobsQueue��Ҳ���Ա�������һ��db�����Caffe2������BlobQueueDB��
```
class BlobsQueueDB : public DB {
  public:
    BlobsQueueDB(...);
    void Close() override {}
    unique_ptr<Cursor> NetCursor() override{...}
    unique_ptr<Transaction> NewTransaction() override {...}
  private:
    std::shared_ptr<BlobsQueue> queue_;
    int key_blob_index_;
    int value_blob_index_;
    float timeout_secs_;
};
```

���⣬Caffe2�����BlobsQueue���������˶Զ��н��д���ġ����������ѳ��õĶ��д���ʽ������ӡ����ӵȣ�����Ϊ������
```
graph TB
    Operator-->|����|CreateBlobsQueueOp
    Operator-->|����|EnqueueBlobsOp
    Operator-->|����|DequeueBlobsOp
    Operator-->|����|CloseBlobsQueueOp
    Operator-->|����|SafeEnqueueBlobsOp
    Operator-->|����|SafeDequeueBlobsOp
    Operator-->|����|WeightedSampleDequeueBlobsOp
```

���⣬Ϊ����֧��һ�ζ�������ӣ�Caffe2�����RebatchingQueue�࣬���ļ�Ҫ�ṹ���£�
```
class RebatchingQueue {
  public:
    bool enqueueOne(...);
    bool enqueueMany(...);
    bool dequeue(...);
  private:
    std::vector<std::vector<TensorCPU>> queue_;
};
```
��BlobsQueue�������������㣬��һ����������queue_�д洢����TensorCPU������Blob*���ڶ���ӵ��EnqueueOne��EnqueueMany������Ӳ�����

��BlobsQueue���ƣ�Caffe2ҲΪRebatchingQueue׼���˶�����д���ġ�����������BlobsQueue���ƣ����ﲻ��׸����

# sgd
������������ݶ��½��йصĲ����������Ͽ��Ը����ļ����²⺬�壬������г��ļ���ǰ׺������Ȥ�Ķ��߿��Բ���Դ�룺
```
adadelta_op
adagrad_op
adam_op
clip_tensor_op
fp16_momentum_sgd_op
fp32_momentum_sgd_op
ftrl_op
gftrl_op
iter_op
lars_op
learning_rate_adaption_op
learning_rate_functors
learning_rate_op
momentum_sgd_op
rmsprop_op
wngrad_op
yellowfin_op
```
�л��������ϸ�ж������е�ϸ�ڡ�

# transform
����coreģ�����������֪��������������Ƕ�ͼ���б任�ķ�������Ҫ����4�֣�
```
//��������������CSE����Tensorflow����
common_subexpression_elimination

//�Ծ���������б任�����Ч��
conv_to_nnpack_transform

//ģʽ�滻��������ʹ�ü򵥵Ľӿڶ���ģʽ�滻����ֻ�趨��һģʽ��ͼ��һ���滻��ͼ����ԭͼ��Ѱ��ģʽ��ͼ��Ȼ���滻Ϊ�滻��ͼ����
pattern_net_transform

//����������ԭ��ת��
single_op_transform
```
��Щ���γ������µļ̳���ϵ��
```
graph TB
    Transform-->|����|CommonSubexpressionEliminationTransform
    Transform-->|����|SingleOpTransform
    Transform-->|����|PatternNetTransform
    SingleOpTransform-->|����|ConvToNNPackTransform
```

# util
Ӧ����ͺ������Ƚ����飬��ʱû��ϸ����

# python
ͨ��ǰ��Ľ��������˽⵽��Caffe2�ĺ��Ĵ�������"C++"ʵ�ֵģ�Ϊ�˷�����python�н��е��ã���Ҫһ�����ߣ�����python����"C++"���롣�����Ĺ����кܶ࣬����boost.python, swig��ctypes��pybind11�ȡ�Caffe2ѡ����pybind11����Ϊ����"C++"11֧�ֵıȽϺã�����API�Ƚϼ򵥡���Tensorflow��pythonǰ�˵���"C++"���ʹ�õ���swig����ʵswig��"C++"11Ҳ��֧�֡��������ѡ�������Ŀǰ��֪ʶ���ǻ��������С�

����Ľӿ��ļ�����_import_c_extention.py�������Ȼ᳢������gpu�汾��Caffe2��ˣ����ʧ���ˣ��᳢������CPU�汾�����У�����CPU��˵ĵ�����ͨ�����µ���䣺
```
from caffe2.python.caffe2_pybind11_state import *
```
��ˣ��ڱ�����ɺ�caffe2/pythonĿ¼�»�����һ����Ϊcaffe2_pybind11_state.so���ļ����ǰ�����Caffe2��"C++"��˵Ķ�̬���ӿ⣬���Ա�python���롣

# contrib
ͬTensorflow��contrib�ļ���һ���������˵��������׵ġ�δ��ʽ����Caffe2��ģ�飬������󲿷ִ�������python�����ġ����Ű汾�������������ȶ�����Щģ����𽥼���Caffe2��pythonģ�顣

# д�ں���
����Tensorflow��Caffe2�ĺ��Ĵ���֮�󣬽�һ���Լ��ĸ��ܡ�
- ����ģ���ԣ�Tensorflow�����ģ�������ķǳ��ã�������ܡ�����ʱ��ͼ��ʾ��ͼ�Ż���op��kernel�����ֵ������������Caffe2�Ĵ����Ե���Щ���ӣ������������ǣ��������Ķ�������һ���ϰ���
- ����淶�ԣ�Tensorflow����Ĺ淶��Ҫ�úܶ࣬��Ȼ���Ĵ����Ƕ��������ɵģ���������ǳ�ͳһ���ļ�ͷ��Э��Ҳ�ǳ�һ�¡�����Caffe2�Ĵ��룬Э����ң�������ͳһ����ƴ���յĸо��Ƚ�ǿ�ң���������ʽ�ϵ����в��㡣
- �ܹ������ԣ�Tensorflow��Ұ�ĺܴ������ռ�Ŀ���Ǳ��һ��ȫ�µġ�����������ͼ����ı�����ԡ����ֱ�����Ի���opԭ�����op��kernel�������ں���������ȷ�����ֿ�����ͬʱ��������ͬһ��������ͼ�Ķ��̲߳���ִ�л��ƣ�Ҳ����CPU��ˮ�ߴ�����ƣ���ˣ�Ӧ��˵�����������ֻ��Tensorflow��һ������Ʒ��������ʵ��ֵԶ��ֹ�ڴˡ�����Caffe2���ܶ������Щ�����ˣ�������redisΪ�н����ֲ�ʽִ�У������ṩ��������Ե�ͬʱ��Ҳ���������ĸ߶ȡ�
��Ȼ������ֻ�Ǹ��˵�һЩ�²⣬�����������룬��Ҳ�ἰʱ���������Լ��Ĺ۵㣬Ҳ��ӭ��������ۡ�