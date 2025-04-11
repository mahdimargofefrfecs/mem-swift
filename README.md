Description:
MEM Swift is an intelligent app designed to enhance the fan experience at sports stadiums. By leveraging artificial intelligence, the app allows fans to order food and beverages directly from their seats. It also provides personalized recommendations based on fan preferences, offering the best kiosks according to factors like price, delivery speed, and distance. The goal is to create a seamless, fast, and customized experience for stadium visitors, in line with the Saudi Vision 2030.


Features:
Food and Beverage Ordering: Fans can browse menus, place orders, and pay for food and drinks directly through the app. Real-time updates allow fans to track their order’s progress and expected delivery time.
AI-Powered Recommendations: The app uses artificial intelligence to recommend kiosks based on fan preferences, including price, delivery speed, and kiosk proximity.
Kiosk Locator: The app uses GPS to show nearby kiosks and help fans choose the most convenient options, ensuring accurate kiosk locations on the map.
Integrated Payment System: Secure and fast payments via an external payment gateway. Multiple payment methods are supported, including credit cards and digital wallets.
Seamless Integration with Stadium Infrastructure: The app works efficiently with existing stadium systems (food kiosks, delivery logistics) to ensure smooth operations and real-time communication.


How It Works:
User Registration: Fans can sign up or log in using their email or social media accounts. Profile customization allows users to set preferences (e.g., food type, delivery speed).
Ordering Process: After selecting a stadium seat, users view available kiosks and menus. AI recommends options based on preferences and location. Fans can place orders, pay, and track them in real-time.
Order Fulfillment: Once placed, the system forwards the order to the nearest kiosk. The kiosk prepares and notifies the system when the order is ready. A delivery person brings the order to the fan’s seat.
Feedback and Rating: After receiving their order, fans can rate the kiosk and delivery experience. The AI learns from feedback to improve future recommendations.


and this the code of the app

import SwiftUI

struct ContentView: View {
    @StateObject var orderManager = OrderManager()
    @State private var isLoggedIn = false
    
    var body: some View {
        NavigationView {
            if isLoggedIn {
                VStack {
                    Text(" قائمة المأكولات والمشروبات")
                        .font(.largeTitle)
                        .bold()
                        .padding()
                    
                    List(menuItems) { item in
                        MenuItemRow(item: item, orderManager: orderManager)
                    }
                    
                    NavigationLink(destination: OrderScreen(orderManager: orderManager)) {
                        Text("🛒 عرض السلة (\(orderManager.totalItems))")
                            .font(.headline)
                            .padding()
                            .background(Color.blue)
                            .foregroundColor(.white)
                            .cornerRadius(10)
                    }
                    .padding()
                }
            } else {
                LoginScreen(isLoggedIn: $isLoggedIn)
            }
        }
    }
}


//
//  LocationManager.swift
//  MEM Swift
//
//  Created by Mahdi Marghalani on 4/10/25.
//


import Foundation
import CoreLocation

class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {
    private var locationManager = CLLocationManager()
    
    @Published var userLocation: CLLocationCoordinate2D?
    
    override init() {
        super.init()
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
        locationManager.startUpdatingLocation()
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        if let location = locations.last {
            DispatchQueue.main.async {
                self.userLocation = location.coordinate
            }
        }
    }
}


import SwiftUI

struct LoginScreen: View {
    @Binding var isLoggedIn: Bool
    @State private var username = ""
    @State private var password = ""

    var body: some View {
        VStack {
            Text("تسجيل الدخول")
                .font(.largeTitle)
                .bold()
                .padding()
            
            TextField("اسم المستخدم", text: $username)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .padding()

            SecureField("كلمة المرور", text: $password)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .padding()

            Button(action: {
                if !username.isEmpty && !password.isEmpty {
                    isLoggedIn = true
                }
            }) {
                Text("تسجيل الدخول")
                    .font(.headline)
                    .padding()
                    .background(Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
            .padding()
        }
    }
}


//
//  MenuItem.swift
//  MEM Swift
//
//  Created by Mahdi Marghalani on 4/11/25.
//


import Foundation

struct MenuItem: Identifiable {
    let id = UUID()
    let name: String
    let price: Double
}

let menuItems = [
    MenuItem(name: "💧 مويه ", price: 2.0),
    MenuItem(name: "🍿 فشار ", price: 5.0),
    MenuItem(name: "🍟 بطاطس مقلية", price: 10.0),
    MenuItem(name: "🥤 مشروب غازي", price: 7.0)
]



//
//  MenuItemRow.swift
//  MEM Swift
//
//  Created by Mahdi Marghalani on 4/11/25.
//


import SwiftUI

struct MenuItemRow: View {
    var item: MenuItem
    @ObservedObject var orderManager: OrderManager
    
    var body: some View {
        HStack {
            Text(item.name)
            Spacer()
            Text("\(item.price, specifier: "%.2f") SAR")
            
            Button(action: {
                orderManager.addItem(item)
            }) {
                Text("+")
                    .font(.headline)
                    .padding()
                    .background(Color.green)
                    .foregroundColor(.white)
                    .cornerRadius(5)
            }
            
            Button(action: {
                orderManager.removeItem(item)
            }) {
                Text("-")
                    .font(.headline)
                    .padding()
                    .background(Color.red)
                    .foregroundColor(.white)
                    .cornerRadius(5)
            }
        }
        .padding()
    }
}


//
//  OrderManager.swift
//  MEM Swift
//
//  Created by Mahdi Marghalani on 4/11/25.
//


import Foundation

class OrderManager: ObservableObject {
    @Published var items: [MenuItem] = []
    
    var totalItems: Int {
        items.count
    }
    
    func addItem(_ item: MenuItem) {
        items.append(item)
    }
    
    func removeItem(_ item: MenuItem) {
        if let index = items.firstIndex(where: { $0.id == item.id }) {
            items.remove(at: index)
        }
    }
}

//
//  OrderScreen.swift
//  MEM Swift
//
//  Created by Mahdi Marghalani on 4/11/25.
//


import SwiftUI

struct OrderScreen: View {
    @ObservedObject var orderManager: OrderManager

    var body: some View {
        VStack {
            Text("🛒 سلة الطلبات")
                .font(.largeTitle)
                .bold()
                .padding()
            
            List {
                ForEach(orderManager.items) { item in
                    HStack {
                        Text(item.name)
                        Spacer()
                        Text("\(item.price, specifier: "%.2f") SAR")
                    }
                }
                .onDelete { indexSet in
                    indexSet.forEach { index in
                        let item = orderManager.items[index]
                        orderManager.removeItem(item)
                    }
                }
            }
            
            Text("الإجمالي: \(orderManager.items.reduce(0) { $0 + $1.price }, specifier: "%.2f") SAR")
                .font(.headline)
                .padding()
        }
    }
}


البيانات المستخدمة اعطيني أبرعه منه بي النسبه ال ٪
