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
    std::unique_ptr<AbstractProduct> CreateObject(const ID &id, Arg &&...arg)

 ��loki��factory������Ա

    AbstractProduct* CreateObject(const IdentifierType& id)
    AbstractProduct* CreateObject(const IdentifierType& id, Parm1 p1)
    AbstractProduct* CreateObject(const IdentifierType& id, Parm1 p1, Parm2 p2)
    ...
    
 �ֱ����ڱ�ʾ��������������������һ�������������������ȵȡ����Կ���������variadic template������һ��ģ���滻����ǰ��n����Ա������

<!--TOC-->
  - [Factory](#factory)
  - [visitor](#visitor)
  - [AbstractFactory](#abstractfactory)
<!--/TOC-->


 ## Factory

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

    bool Register(const IdentifierType &id, const std::function<std::unique_ptr<AbstractProduct> (Param...)>& creator)
creator ���������������functor, �ܽ��ܲ���Param...����������Ϊstd::unique_ptr\<AbstractProduct>��


      template<class ID, class... Arg>
      std::unique_ptr<AbstractProduct> CreateObject(const ID &id, Arg &&...arg);

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
      Product(int xa, int ya) : x(std::move(xa)), y(std::move(ya)) {}
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

    std::unique_ptr<Product >createProductNull()
    {
      cout << "createProductNull()" << endl;
      return std::make_unique<Product>();
    }
    auto createProductParm(int a, int b)
    {
      cout << "createProductParm( int a, int b ) " << endl;
      return std::make_unique<Product>(a,b);
    }

    ///////////////////////////////////////////////////
    // Overloaded creator functions
    ///////////////////////////////////////////////////
    auto createProductNullOver()
    {
      cout << "createProductOver()" << endl;
      return std::make_unique<Product>();
    }
    auto createProductParmOver(int a, int b)
    {
      cout << "createProductOver( int a, int b )" << endl;
      return std::make_unique<Product>(a,b);
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
    
## visitor

 visitorģʽ��һ�����ģʽ���������ڲ��ı����ݽṹ������£�Ϊ���ݽṹ�е�Ԫ������µĲ�����visitorģʽ��Ŀ���ǽ����ݽṹ�����ݲ������롣visitorģʽ���ŵ��������µĲ��������ף���Ϊ�����µĲ�������ζ������һ���µ�visitor��

��Ҫʵ��Acyclic Visitor����ʹ�� Basevisitor��Ϊ"strawman" base class�� visitor��visitable:

    class BaseVisitor;

    template<typename R, bool constparam, class... Head>
    class Visitor;

    template<
      typename R = void,
      bool ConstVisitable = false,
      template<typename, class> class CatchAll = DefaultCatchAll>
    class BaseVisitable;

Visitor��BaseVisitable�ĵ�һ��template�����ֱ��ǳ�Ա����Visit()�ķ���ֵ���ͺ�BaseVisitable�ķ���ֵ���͡��ڶ�������ָ��Visit()�Ĳ����Ƿ�Ϊconst���á�
BaseVisitable�ĵ�����template�����Ǹ�policy����������catch-all����

��BaseVisitable��������ļ̳���ϵ��root class,Ȼ��������̳���ϵ��ÿһ��class��ʹ�ú�DEFINE_VISITABLE()��

��Visitor<R,ConstVisitable ,Product... >�����������concrete visitor classes�������Product����Ĳ�Ʒ�̳���ϵ������е����ͣ�����Լ̳���ϵ�е�ÿһ����ʵ������Ա���� Visit:

    class VariableVisitor : public Loki::Visitor < void
      , false,Shape, Circle > {
     public:
      void Visit(Shape&) { std::cout << "void Visit(Shape&)\n"; }
      void Visit(Circle&) { std::cout << "void Visit(Circle&)\n"; }
    };


 ����:
 
    #include <loki/Visitor.h>
    #include <iostream>

    class Shape : public Loki::BaseVisitable<>
    {
     public:
      LOKI_DEFINE_VISITABLE()
    };

    class Circle : public Shape
    {
     public:
      LOKI_DEFINE_VISITABLE()
    };

    class VariableVisitor : public Loki::Visitor < void
      , false,Shape, Circle > {
     public:
      void Visit(Shape&) { std::cout << "void Visit(Shape&)\n"; }
      void Visit(Circle&) { std::cout << "void Visit(Circle&)\n"; }
    };

    class CShape : public Loki::BaseVisitable<void,true>
    {
     public:
      LOKI_DEFINE_CONST_VISITABLE()
    };

    class CCircle : public CShape
    {
     public:
      LOKI_DEFINE_CONST_VISITABLE()
    };

    class CVariableVisitor : public Loki::Visitor<void,true, CShape,CCircle>
    {
     public:
      void Visit(const CShape&) { std::cout << "void Visit(CShape&)\n"; }
      void Visit(const CCircle&) { std::cout << "void Visit(CCircle&)\n"; }
    };

    int main()
    {
      VariableVisitor visitor;
      Circle           circle;
      Shape*           dyn = &circle;
      Shape           shape ;
      dyn->Accept(visitor);
      dyn = &shape;
      dyn->Accept(visitor);

      CVariableVisitor cvisitor;
      CCircle           ccircle;
      CShape*           cdyn = &ccircle;
      cdyn->Accept(cvisitor);
      CShape cshape;
      cdyn = &cshape;
      cdyn->Accept(cvisitor);
    }

## AbstractFactory


    template<class... U>
    using AbstractFactory = ...;

����U...������������ָ�����Factory��Ҫ���ɵ�abstract products������

using AbstractEnemyFactory=AbstractFactory<Soldier, Monster>;

AbstractFactory�ṩһ����ΪCreate()�ĳ�Ա������������abstract products�е�һ��������ʵ���������ڴ���һ��abstract product������

    AbstractEnemyFactory factory;
    auto soldier = factory.Create<Soldier>();
    auto monster = factory.Create<Monster>();

  Ϊʵ��AbstractFactory������Ľӿڣ�Loki�ṩ��һ��concreteFactory template����������:

    template<class AbstractFact, 
    class TList = typename AbstractFact::ProductList>
      using ConcreteFactory = ...;
    
   ����AbstractFact�ǽ���ʵ������֮AbstractFactoryģ���ʵ���������ĵ�AbstractEnemyFactory��TList��concrete products typelist����mp::mp_list<SillySoldier,SillyMonster>��

   ����:

    #include <iostream>
    #include <string>
    #include <memory>
    #include <loki/AbstractFactory.h>


    using namespace Loki;
    using std::cout;
    using std::endl;

    struct Soldier
    {
    public:
      virtual ~Soldier(){};

      virtual void print() const {
      
        cout << "Soldier\n";
      }
    };

    struct SillySoldier:public Soldier
    {
    public:
      virtual void print() const {
        cout << "SillySoldier\n";
      }
    };

    struct Monster
    {
    public:
      virtual ~Monster() =default;
      virtual void print() const = 0;
    };

    struct SillyMonster:public Monster
    {
    public:
      virtual void print() const {
        cout << "SillyMonster\n";
      }
    };
    using AbstractEnemyFactory=AbstractFactory<Soldier, Monster>;

    using ConcreteEnemyFactory= ConcreteFactory<AbstractEnemyFactory, mp::mp_list<SillySoldier,SillyMonster>> ;
    int main() {
      ConcreteEnemyFactory cef;
      AbstractEnemyFactory *pA = &cef;
      auto pS = pA->Create<Soldier>();
      std::unique_ptr<Soldier> p(pS);
      p->print();
    }

