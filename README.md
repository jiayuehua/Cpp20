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
���ӣ�

    #include <iostream>
    #include <string>
    #include <loki/Factory.h>


    using namespace Loki;
    using std::cout;
    using std::endl;


    ////////////////////////////////////////////
    // Object to create: Product
    // Constructor with 0 and 2 arguments
    ////////////////////////////////////////////

    class AbstractProduct
    {
    public:
      virtual int get_x() const = 0;
      virtual int get_y() const = 0;
      virtual ~AbstractProduct() = default;
    };

    class Product : public AbstractProduct
    {
      int x = 0;
      int y = 0;

    public:
      Product() = default;
      Product(int xa, int ya) : x(xa), y(ya) {}
      int get_x() const
      {
        return x;
      }
      int get_y() const
      {
        return y;
      }
    };

    ///////////////////////////////////////////////////////////
    // Factory for creation a Product object without parameters
    ///////////////////////////////////////////////////////////

    typedef Factory<int, DefaultFactoryError, AbstractProduct>
      PFactoryNull;

    /////////////////////////////////////////////////////////////
    // Factory for creation a Product object with 2 parameters
    /////////////////////////////////////////////////////////////

    typedef Factory<int, DefaultFactoryError, AbstractProduct, int, int>
      PFactory;

    ////////////////////////////////////////////////////
    // Creator functions with different names
    ////////////////////////////////////////////////////

    Product *createProductNull()
    {
      cout << "createProductNull()" << endl;
      return new Product;
    }
    Product *createProductParm(int a, int b)
    {
      cout << "createProductParm( int a, int b ) " << endl;
      return new Product(a, b);
    }



    void testFactoryNull()
    {
      PFactoryNull pFactoryNull;
      bool const ok1 = pFactoryNull.Register(1, createProductNull);
      if (ok1) {
        std::unique_ptr<AbstractProduct> pObject(pFactoryNull.CreateObject(1));
        if (pObject.get()) {
          cout << "pObject->get_x() = " << pObject->get_x() << endl;
          cout << "pObject->get_y() = " << pObject->get_y() << endl;
        }
      }
    }

    void testFactoryBinary()
    {
      PFactory pFactoryNull;
      bool const ok1 = pFactoryNull.Register(1, createProductParm);
      if (ok1) {
        std::unique_ptr<AbstractProduct> pObject(pFactoryNull.CreateObject(1, 5, 5));
        if (pObject) {
          cout << "pObject->get_x() = " << pObject->get_x() << endl;
          cout << "pObject->get_y() = " << pObject->get_y() << endl;
        }
      }
    }

    int main()
    {
      testFactoryNull();
      testFactoryBinary();
    }
    
 �����

      createProductNull()
      pObject->get_x() = 0
      pObject->get_y() = 0
      createProductParm( int a, int b ) 
      pObject->get_x() = 5
      pObject->get_y() = 5
    
   

  
