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








