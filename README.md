# Loki11 lib
update andrei's loki lib to C++11
��Ҫ�Ľ�
- �ӿڸ�Ϊʹ��variadic templates��������typelist��ʵ�����õ�typelist�ĵط���Ϊʹ��boost mp11���mp_list
- typelist�㷨��Ϊʹ��mp11���㷨��
- ��typetraits���ָ�Ϊֱ��ʹ��C++11��type_traits
- ��functorɾ������Ϊʹ��C++11��std::function

��Ҫ�Ա�loki11��loki
��factoryΪ��˵����Loki11���������������ֻ��һ����Աģ��

    template<class ID, class... Arg>
    AbstractProduct *CreateObject(const ID &id, Arg &&...arg)

 ��loki��factory������Ա

    AbstractProduct* CreateObject(const IdentifierType& id)
    AbstractProduct* CreateObject(const IdentifierType& id, Parm1 p1)
    AbstractProduct* CreateObject(const IdentifierType& id, Parm1 p1, Parm2 p2)
    ...
    
 �ֱ����ڱ�ʾ��������������������һ�������������������ȵȡ����Կ���������variadic template������һ��ģ���滻����ǰ��n����Ա������

 ## Factoryʹ�÷���

 Factoryģ��λ��loki/Factory.h
 ����

    template<
      typename IdentifierType,
      template<typename, class>
      class FactoryErrorPolicy,
      class AbstractProduct,
      class... Param>
    class Factory ;

�������Ϊĳһ�̳���ϵ�ṩobject factories,���������е�AbstractProduct Ӧָ��Ϊ�ü̳���ϵ��base class.

IdentifierType ������ʶ�̳���ϵ�е�ĳ���ͱ�IdentifierType�����������ͱ�(ordered Type)������Ϊstd::map��key���ͣ����õ�������int, std::string��

FactoryErrorPolicy����ָ��������Ĳ��ԡ�

Factoryʵ�������»�������

    bool Register(const IdentifierType &id, const std::function<AbstractProduct *(Param...)>& creator)
creator ���������������functor, �ܽ��ܲ���Param...����������ΪAbstractProductָ�롣


      template<class ID, class... Arg>
      AbstractProduct *CreateObject(const ID &id, Arg &&...arg);

���Ϻ�����ѯid�Ƿ�ע��������ע����Ļ�������id��Ӧ��creator����������Ϊarg...���������󣬷��ؽ����








