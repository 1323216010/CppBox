```cpp
#include <iostream>
#include <map>
#include <string>
#include <typeinfo>

class Any {
private:
    struct AnyBase {
        virtual ~AnyBase() {}
        virtual AnyBase* clone() const = 0;
    };

    template <typename ValueType>
    struct AnyDerived : public AnyBase {
        AnyDerived(const ValueType& value) : held(value) {}
        AnyBase* clone() const override {
            return new AnyDerived(held);
        }
        ValueType held;
    };

public:
    Any() : content(nullptr) {}

    template <typename ValueType>
    Any(const ValueType& value)
        : content(new AnyDerived<ValueType>(value)) {}

    Any(const Any& other)
        : content(other.content ? other.content->clone() : nullptr) {}

    Any& operator=(const Any& other) {
        if (content != other.content) {
            delete content;
            content = other.content ? other.content->clone() : nullptr;
        }
        return *this;
    }

    ~Any() {
        delete content;
    }

    template <typename ValueType>
    ValueType& cast() const {
        auto derived = static_cast<AnyDerived<ValueType>*>(content);
        return derived->held;
    }

private:
    AnyBase* content;
};

int main() {
    std::map<std::string, Any> myMap;

    // 存储不同类型的键值对
    myMap["name"] = std::string("Alice");
    myMap["age"] = 25;
    myMap["isStudent"] = true;

    // 从map中获取值并进行类型转换
    std::string name = myMap["name"].cast<std::string>();
    int age = myMap["age"].cast<int>();
    bool isStudent = myMap["isStudent"].cast<bool>();

    // 输出值
    std::cout << "Name: " << name << std::endl;
    std::cout << "Age: " << age << std::endl;
    std::cout << "Is Student: " << (isStudent ? "Yes" : "No") << std::endl;

    return 0;
}

```

