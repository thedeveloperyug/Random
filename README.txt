#ifndef FUNCTIONALITIES_H
#define FUNCTIONALITIES_H

#include "Brand.h"
#include <vector>
#include <memory>
#include "ContainerEmptyDataException.h"
using BrandPtr = std::shared_ptr<Brand>;
using BrandContainer = std::vector<BrandPtr>;
using RegistrationCost = std::vector<float>;
using CarContainer = std::vector<Car>;
using RefCarContainer = std::vector<CarRef>;

void CreateBrandObjects(BrandContainer& brand);
RegistrationCost BrandLicenseRegistrationCost(const BrandContainer&brand);
std::optional<RefCarContainer> CarHavingGivenBrandType(const BrandContainer&brand, const BrandType &type);
std::optional<CarContainer> BrandPriceABoveThresold(const BrandContainer&brand, float thrseold);
void DetailsChassisMinimumPrice(const BrandContainer& brand);
void CarBrandHavingSeatCountAbove4(const BrandContainer& brand);

#endif // FUNCTIONALITIES_H

#include "Functionalities.h"

void CreateBrandObjects(BrandContainer &brand)
{
    Car car1 = Car(4,233000.0f,340.8f,CarChassisType::CARBON_FIBER);
    Car car2 = Car(2,123000.0f,134.8f,CarChassisType::STEEL);
    Car car3 = Car(3,22000.0f,324.8f,CarChassisType::CARBON_FIBER);
    CarRefContainer Ref1 {car1,car2,car3};
    brand.emplace_back(std::make_shared<Brand>(BrandType::CONSUMER,"MH123",Ref1));

    Car car4 = Car(2,23000.0f,340.8f,CarChassisType::STEEL);
    Car car5 = Car(2,220300.0f,140.8f,CarChassisType::STEEL);
    Car car6 = Car(3,23000.0f,340.8f,CarChassisType::CARBON_FIBER);
    CarRefContainer Ref2 {car4,car5,car6};
    brand.emplace_back(std::make_shared<Brand>(BrandType::SPECIAL_PURPOSE,"MH124",Ref2));

    Car car7 = Car(2,12000.0f,340.8f,CarChassisType::STEEL);
    Car car8 = Car(4,22000.0f,140.8f,CarChassisType::CARBON_FIBER);
    Car car9 = Car(3,253000.0f,340.8f,CarChassisType::CARBON_FIBER);
    CarRefContainer Ref3 {car7,car8,car9};
    brand.emplace_back(std::make_shared<Brand>(BrandType::TRANSPORT,"MH125",Ref3));
}

RegistrationCost BrandLicenseRegistrationCost(const BrandContainer &brand)
{
    if(brand.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }
    
    float sum{0.0f};
    RegistrationCost result;
    Car p;
    for(BrandPtr ptr: brand){
        sum = 0.0f;
        if(ptr){
            
            for(CarRef ref: ptr->ref()){
                 p = ref.get();
                //  sum+= ((p.price())*(p.wheelBase())*0.01);
                sum = p.price();
            }
            result.push_back(sum);
        }
    }

    if(result.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }

    return result;
}

std::optional<RefCarContainer> CarHavingGivenBrandType(const BrandContainer &brand, const BrandType &type)
{
    if(brand.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }

    RefCarContainer result;
    Car p;
    for(BrandPtr ptr: brand){
        if(ptr && ptr->type()==type){
             for(CarRef ref : ptr->ref()){
                p = ref.get();
                result.push_back(std::ref(p));
             }
        }
    }

    if(result.empty()){
        throw ContainerEmptyException("Data is EMpty",std::future_errc::no_state);
    }

    return result;
}

std::optional<CarContainer> BrandPriceABoveThresold(const BrandContainer &brand, float thrseold)
{
     if(brand.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }

    CarContainer result;
    Car p;
    for(BrandPtr ptr: brand){
        if(ptr){
            for(CarRef ref: ptr->ref()){
                if(ref.get().price()>thrseold){
                      p = ref.get();
                      result.push_back(p);
                }
            }
        }
    }

    if(result.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }

    return result ;
}

void DetailsChassisMinimumPrice(const BrandContainer &brand)
{
     if(brand.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }

    float MinPrice {0.0f};
    for(CarRef ref: brand[0]->ref()){
        MinPrice = ref.get().price();
    }
    
    for(BrandPtr ptr: brand){
        for(CarRef ref: ptr->ref()){
             if(MinPrice>ref.get().price()){
                MinPrice = ref.get().price();
             }
        }
    }

    CarChassisType type;

    for(BrandPtr ptr: brand){
        for(CarRef ref: ptr->ref()){
             if(MinPrice==ref.get().price()){
                type = ref.get().chassis();
                break;
             }
        }
    }
    
    std::string val = "";
    if(type==CarChassisType::CARBON_FIBER){
        val = "CARBON_FiBER";
    }
    else if(type==CarChassisType::STEEL){
        val = "STEEL";
    }
    
    std::cout<<"Car Having Minimum Price with its chassis type "<<val<<std::endl;

}

void CarBrandHavingSeatCountAbove4(const BrandContainer &brand)
{
    if(brand.empty()){
        throw ContainerEmptyException("Data is empty",std::future_errc::no_state);
    }
    int count{0};
    bool flag{true};
    int seatCount = 4;
     for(BrandPtr ptr: brand){
        flag = true;
        for(CarRef ref: ptr->ref()){
            if(ref.get().seatCount()<seatCount){
                flag = false;
                break;
            }
        }
        count++;
        if(flag){
            std::cout<<"Brand "<<count<<" Have All Cars Seat COUNT Above 4"<<std::endl;
        }
        else{
            std::cout<<"Brand "<<count<<" Have not All the Cars Seat COUNT Above 4"<<std::endl;
        }
    }
}












#include "Functionalities.h"

int main(){
    BrandContainer brand;
    CreateBrandObjects(brand);
    BrandType type = BrandType::CONSUMER;
    std::future<RegistrationCost> cost = std::async(&BrandLicenseRegistrationCost,std::ref(brand));

    std::future<std::optional<RefCarContainer>> costref = std::async(&CarHavingGivenBrandType,std::ref(brand),std::ref(type));   

    std::future<std::optional<CarContainer>> cont = std::async(&BrandPriceABoveThresold,std::ref(brand),10000);
    
    std::future<void> Chassis = std::async(&DetailsChassisMinimumPrice,std::ref(brand));

    std::future<void> Having = std::async(& CarBrandHavingSeatCountAbove4,std::ref(brand));

    try
    {
        RegistrationCost CostReg = cost.get();
        int count{0};
        for(float reg: CostReg){
            std::cout<<"Brand License Registration Cost "<<count++ <<" "<<reg<<std::endl;
        }

        std::optional<RefCarContainer> ref = costref.get();
        if(ref.has_value()){
            RefCarContainer val = ref.value();

            for(CarRef re: val){
                std::cout<<re.get()<<std::endl;
            }
        }

        std::optional<CarContainer> conti = cont.get();
        if(conti.has_value()){
            CarContainer cc = conti.value();
            for(Car c: cc){
                std::cout<<c<<std::endl;
            }
        }

        Chassis.get();
        Having.get();

    }
    catch(ContainerEmptyException& e)
    {
        std::cerr << e.what() << '\n';
    }
    
}









