# Mobile App Testing 📱

## What is Mobile App Testing?

Mobile app testing validates applications across different mobile devices, operating systems, networks, and usage scenarios. It encompasses functionality, performance, usability, security, and device-specific features testing to ensure optimal user experience across diverse mobile environments.

Mobile testing addresses unique challenges including device fragmentation, network variability, battery optimization, touch interfaces, sensor integration, and platform-specific behaviors that don't exist in traditional web or desktop applications.

## Key Concepts

### Types of Mobile Testing
- **Native App Testing**: Platform-specific applications (iOS/Android)
- **Hybrid App Testing**: Cross-platform applications using web technologies
- **Web App Testing**: Mobile-optimized web applications
- **Progressive Web App (PWA) Testing**: Web apps with native-like features
- **Cross-Platform Testing**: React Native, Flutter, Xamarin applications

### Device Testing Approaches
- **Real Device Testing**: Testing on physical devices
- **Emulator/Simulator Testing**: Virtual device testing
- **Cloud Device Testing**: Remote device lab testing
- **Crowd Testing**: Distributed testing across user devices

### Mobile-Specific Testing Areas
- **Touch and Gesture Testing**: Swipe, pinch, zoom, tap interactions
- **Sensor Testing**: GPS, accelerometer, camera, microphone
- **Network Testing**: 3G, 4G, 5G, WiFi, offline scenarios
- **Battery and Performance Testing**: Resource usage optimization
- **Security Testing**: App permissions, data encryption, secure storage

## Tools & Frameworks

### Appium Mobile Automation
```python
# Appium test automation framework
from appium import webdriver
from appium.webdriver.common.touch_action import TouchAction
from appium.webdriver.common.mobileby import MobileBy
import pytest
import time

class MobileTestBase:

    def setup_android_driver(self):
        """Setup Android driver configuration"""
        desired_caps = {
            'platformName': 'Android',
            'platformVersion': '12.0',
            'deviceName': 'Android Emulator',
            'app': '/path/to/your/app.apk',
            'automationName': 'UiAutomator2',
            'appPackage': 'com.example.myapp',
            'appActivity': '.MainActivity',
            'newCommandTimeout': 300,
            'autoGrantPermissions': True
        }

        self.driver = webdriver.Remote(
            'http://localhost:4723/wd/hub',
            desired_caps
        )
        return self.driver

    def setup_ios_driver(self):
        """Setup iOS driver configuration"""
        desired_caps = {
            'platformName': 'iOS',
            'platformVersion': '16.0',
            'deviceName': 'iPhone 14',
            'app': '/path/to/your/app.ipa',
            'automationName': 'XCUITest',
            'bundleId': 'com.example.myapp',
            'newCommandTimeout': 300,
            'autoAcceptAlerts': True
        }

        self.driver = webdriver.Remote(
            'http://localhost:4723/wd/hub',
            desired_caps
        )
        return self.driver

class TestMobileAppFunctionality(MobileTestBase):

    def test_app_launch_and_navigation(self):
        """Test app launch and basic navigation"""
        driver = self.setup_android_driver()

        try:
            # Verify app launches successfully
            assert driver.current_activity == '.MainActivity'

            # Test navigation to different screens
            menu_button = driver.find_element(MobileBy.ID, 'com.example.myapp:id/menu_button')
            menu_button.click()

            # Verify menu opens
            menu_drawer = driver.find_element(MobileBy.ID, 'com.example.myapp:id/navigation_drawer')
            assert menu_drawer.is_displayed()

            # Navigate to profile screen
            profile_item = driver.find_element(MobileBy.XPATH, "//android.widget.TextView[@text='Profile']")
            profile_item.click()

            # Verify profile screen loads
            profile_header = driver.find_element(MobileBy.ID, 'com.example.myapp:id/profile_header')
            assert profile_header.is_displayed()

        finally:
            driver.quit()

    def test_user_registration_flow(self):
        """Test complete user registration workflow"""
        driver = self.setup_android_driver()

        try:
            # Navigate to registration screen
            register_button = driver.find_element(MobileBy.ID, 'com.example.myapp:id/register_button')
            register_button.click()

            # Fill registration form
            test_data = {
                'first_name': 'John',
                'last_name': 'Doe',
                'email': 'john.doe@example.com',
                'password': 'SecurePass123!',
                'confirm_password': 'SecurePass123!'
            }

            for field_name, value in test_data.items():
                field = driver.find_element(MobileBy.ID, f'com.example.myapp:id/{field_name}')
                field.clear()
                field.send_keys(value)

            # Scroll to submit button if needed
            driver.execute_script('mobile: scroll', {'direction': 'down'})

            # Submit registration
            submit_button = driver.find_element(MobileBy.ID, 'com.example.myapp:id/submit_button')
            submit_button.click()

            # Verify registration success
            success_message = driver.find_element(MobileBy.ID, 'com.example.myapp:id/success_message')
            assert 'Registration successful' in success_message.text

        finally:
            driver.quit()

    def test_touch_gestures(self):
        """Test various touch gestures and interactions"""
        driver = self.setup_android_driver()

        try:
            # Test swipe gesture
            self.perform_swipe_test(driver)

            # Test pinch and zoom
            self.perform_pinch_zoom_test(driver)

            # Test long press
            self.perform_long_press_test(driver)

            # Test multi-touch
            self.perform_multi_touch_test(driver)

        finally:
            driver.quit()

    def perform_swipe_test(self, driver):
        """Test swipe gestures"""
        # Navigate to swipeable content
        image_gallery = driver.find_element(MobileBy.ID, 'com.example.myapp:id/image_gallery')

        # Get element dimensions
        size = image_gallery.size
        location = image_gallery.location

        # Perform horizontal swipe (left to right)
        start_x = location['x'] + size['width'] * 0.8
        start_y = location['y'] + size['height'] * 0.5
        end_x = location['x'] + size['width'] * 0.2
        end_y = start_y

        action = TouchAction(driver)
        action.press(x=start_x, y=start_y).wait(1000).move_to(x=end_x, y=end_y).release().perform()

        time.sleep(2)

        # Verify swipe action worked (e.g., image changed)
        current_image = driver.find_element(MobileBy.ID, 'com.example.myapp:id/current_image')
        assert current_image.get_attribute('content-desc') != 'Image 1'

    def perform_pinch_zoom_test(self, driver):
        """Test pinch and zoom gestures"""
        # Navigate to zoomable content
        map_view = driver.find_element(MobileBy.ID, 'com.example.myapp:id/map_view')

        # Perform zoom in (pinch out)
        action1 = TouchAction(driver)
        action2 = TouchAction(driver)

        # Define zoom gesture coordinates
        size = map_view.size
        location = map_view.location

        center_x = location['x'] + size['width'] / 2
        center_y = location['y'] + size['height'] / 2

        # Zoom in gesture
        action1.press(x=center_x - 50, y=center_y).wait(1000).move_to(x=center_x - 100, y=center_y).release()
        action2.press(x=center_x + 50, y=center_y).wait(1000).move_to(x=center_x + 100, y=center_y).release()

        from appium.webdriver.common.multi_action import MultiAction
        multi_action = MultiAction(driver)
        multi_action.add(action1, action2)
        multi_action.perform()

        time.sleep(2)

        # Verify zoom level changed
        zoom_level = driver.find_element(MobileBy.ID, 'com.example.myapp:id/zoom_level')
        assert float(zoom_level.text) > 1.0
```

### Real Device Testing Setup
```python
# Real device testing configuration
class RealDeviceTestSuite:

    def __init__(self):
        self.device_farm = {
            'android_devices': [
                {'model': 'Samsung Galaxy S21', 'os': '12.0', 'udid': 'device1_udid'},
                {'model': 'Google Pixel 6', 'os': '13.0', 'udid': 'device2_udid'},
                {'model': 'OnePlus 9', 'os': '11.0', 'udid': 'device3_udid'}
            ],
            'ios_devices': [
                {'model': 'iPhone 13', 'os': '16.0', 'udid': 'device4_udid'},
                {'model': 'iPhone 12 Mini', 'os': '15.0', 'udid': 'device5_udid'},
                {'model': 'iPad Pro', 'os': '16.0', 'udid': 'device6_udid'}
            ]
        }

    def setup_device_capabilities(self, device_info):
        """Setup capabilities for specific device"""
        if device_info['model'].startswith('iPhone') or device_info['model'].startswith('iPad'):
            return {
                'platformName': 'iOS',
                'platformVersion': device_info['os'],
                'deviceName': device_info['model'],
                'udid': device_info['udid'],
                'automationName': 'XCUITest',
                'bundleId': 'com.example.myapp',
                'newCommandTimeout': 300
            }
        else:
            return {
                'platformName': 'Android',
                'platformVersion': device_info['os'],
                'deviceName': device_info['model'],
                'udid': device_info['udid'],
                'automationName': 'UiAutomator2',
                'appPackage': 'com.example.myapp',
                'appActivity': '.MainActivity',
                'newCommandTimeout': 300
            }

    def test_across_devices(self, test_function):
        """Execute test function across multiple devices"""
        results = {}

        all_devices = self.device_farm['android_devices'] + self.device_farm['ios_devices']

        for device in all_devices:
            try:
                capabilities = self.setup_device_capabilities(device)
                driver = webdriver.Remote('http://localhost:4723/wd/hub', capabilities)

                # Execute test function
                test_result = test_function(driver)

                results[device['model']] = {
                    'status': 'PASS' if test_result else 'FAIL',
                    'details': test_result
                }

                driver.quit()

            except Exception as e:
                results[device['model']] = {
                    'status': 'ERROR',
                    'error': str(e)
                }

        return results

    def test_device_compatibility(self, driver):
        """Test app compatibility on specific device"""
        compatibility_checks = {
            'app_launches': False,
            'ui_renders_correctly': False,
            'touch_responsive': False,
            'orientation_works': False,
            'performance_acceptable': False
        }

        try:
            # Test app launch
            driver.wait_activity('.MainActivity', timeout=30)
            compatibility_checks['app_launches'] = True

            # Test UI rendering
            main_content = driver.find_element(MobileBy.ID, 'com.example.myapp:id/main_content')
            compatibility_checks['ui_renders_correctly'] = main_content.is_displayed()

            # Test touch responsiveness
            test_button = driver.find_element(MobileBy.ID, 'com.example.myapp:id/test_button')
            test_button.click()
            time.sleep(1)

            response_element = driver.find_element(MobileBy.ID, 'com.example.myapp:id/response_text')
            compatibility_checks['touch_responsive'] = response_element.is_displayed()

            # Test orientation change
            driver.orientation = 'LANDSCAPE'
            time.sleep(2)

            landscape_content = driver.find_element(MobileBy.ID, 'com.example.myapp:id/main_content')
            compatibility_checks['orientation_works'] = landscape_content.is_displayed()

            driver.orientation = 'PORTRAIT'

            # Test basic performance
            start_time = time.time()
            driver.find_element(MobileBy.ID, 'com.example.myapp:id/heavy_operation_button').click()

            # Wait for operation to complete
            driver.find_element(MobileBy.ID, 'com.example.myapp:id/operation_complete', timeout=10)
            end_time = time.time()

            operation_time = end_time - start_time
            compatibility_checks['performance_acceptable'] = operation_time < 5.0

        except Exception as e:
            print(f"Compatibility test error: {e}")

        return compatibility_checks
```

### GPS and Location Testing
```python
# GPS and location testing
class LocationTestSuite:

    def test_gps_functionality(self, driver):
        """Test GPS and location services"""
        # Set mock location for testing
        driver.set_location(latitude=37.7749, longitude=-122.4194, altitude=100)

        # Navigate to location-based feature
        location_button = driver.find_element(MobileBy.ID, 'com.example.myapp:id/get_location_button')
        location_button.click()

        # Wait for location to be detected
        time.sleep(5)

        # Verify location is displayed
        location_display = driver.find_element(MobileBy.ID, 'com.example.myapp:id/location_display')
        location_text = location_display.text

        # Check if location coordinates are reasonable
        assert '37.77' in location_text  # Latitude
        assert '122.41' in location_text  # Longitude

    def test_location_permissions(self, driver):
        """Test location permission handling"""
        # Trigger location permission request
        location_feature = driver.find_element(MobileBy.ID, 'com.example.myapp:id/location_feature')
        location_feature.click()

        # Handle permission dialog
        try:
            # For Android
            allow_button = driver.find_element(MobileBy.ID, 'com.android.permissioncontroller:id/permission_allow_button')
            allow_button.click()
        except:
            try:
                # For iOS
                allow_button = driver.find_element(MobileBy.XPATH, "//XCUIElementTypeButton[@name='Allow']")
                allow_button.click()
            except:
                pass  # Permission might already be granted

        # Verify location functionality works after permission granted
        time.sleep(3)
        location_status = driver.find_element(MobileBy.ID, 'com.example.myapp:id/location_status')
        assert 'Location enabled' in location_status.text

    def test_offline_location_handling(self, driver):
        """Test location handling when offline"""
        # Disable network connectivity
        driver.set_network_connection(0)  # Airplane mode

        # Try to get location
        get_location_button = driver.find_element(MobileBy.ID, 'com.example.myapp:id/get_location_button')
        get_location_button.click()

        # Verify appropriate error handling
        time.sleep(5)
        error_message = driver.find_element(MobileBy.ID, 'com.example.myapp:id/location_error')
        assert 'Unable to get location' in error_message.text or 'Network unavailable' in error_message.text

        # Re-enable network
        driver.set_network_connection(6)  # WiFi + Data
```

### Permission Testing Framework
```python
# Mobile app permissions testing
class PermissionTestSuite:

    def __init__(self, driver):
        self.driver = driver
        self.permission_scenarios = {
            'camera': {
                'trigger_element': 'com.example.myapp:id/camera_button',
                'android_permission': 'android.permission.CAMERA',
                'ios_permission': 'Camera'
            },
            'microphone': {
                'trigger_element': 'com.example.myapp:id/record_button',
                'android_permission': 'android.permission.RECORD_AUDIO',
                'ios_permission': 'Microphone'
            },
            'location': {
                'trigger_element': 'com.example.myapp:id/location_button',
                'android_permission': 'android.permission.ACCESS_FINE_LOCATION',
                'ios_permission': 'Location'
            },
            'contacts': {
                'trigger_element': 'com.example.myapp:id/contacts_button',
                'android_permission': 'android.permission.READ_CONTACTS',
                'ios_permission': 'Contacts'
            },
            'storage': {
                'trigger_element': 'com.example.myapp:id/file_picker_button',
                'android_permission': 'android.permission.READ_EXTERNAL_STORAGE',
                'ios_permission': 'Photos'
            }
        }

    def test_permission_request_flow(self, permission_type):
        """Test permission request and handling"""
        scenario = self.permission_scenarios[permission_type]

        # Trigger permission request
        trigger_element = self.driver.find_element(MobileBy.ID, scenario['trigger_element'])
        trigger_element.click()

        # Handle permission dialog
        permission_result = self.handle_permission_dialog(permission_type)

        # Verify app behavior based on permission result
        if permission_result == 'granted':
            self.verify_feature_works(permission_type)
        else:
            self.verify_permission_denied_handling(permission_type)

        return permission_result

    def handle_permission_dialog(self, permission_type):
        """Handle permission dialog interaction"""
        time.sleep(2)  # Wait for dialog to appear

        try:
            platform = self.driver.capabilities['platformName'].lower()

            if platform == 'android':
                return self.handle_android_permission()
            else:
                return self.handle_ios_permission()

        except Exception as e:
            print(f"Permission dialog handling error: {e}")
            return 'error'

    def handle_android_permission(self):
        """Handle Android permission dialogs"""
        try:
            # Check for permission dialog
            allow_button = self.driver.find_element(
                MobileBy.ID, 
                'com.android.permissioncontroller:id/permission_allow_button'
            )
            allow_button.click()
            return 'granted'

        except:
            try:
                deny_button = self.driver.find_element(
                    MobileBy.ID, 
                    'com.android.permissioncontroller:id/permission_deny_button'
                )
                deny_button.click()
                return 'denied'
            except:
                return 'no_dialog'

    def handle_ios_permission(self):
        """Handle iOS permission dialogs"""
        try:
            # Check for permission alert
            allow_button = self.driver.find_element(
                MobileBy.XPATH, 
                "//XCUIElementTypeButton[@name='Allow']"
            )
            allow_button.click()
            return 'granted'

        except:
            try:
                dont_allow_button = self.driver.find_element(
                    MobileBy.XPATH, 
                    "//XCUIElementTypeButton[@name=\"Don't Allow\"]"
                )
                dont_allow_button.click()
                return 'denied'
            except:
                return 'no_dialog'

    def verify_feature_works(self, permission_type):
        """Verify feature works when permission is granted"""
        time.sleep(3)

        verification_elements = {
            'camera': 'com.example.myapp:id/camera_preview',
            'microphone': 'com.example.myapp:id/recording_indicator',
            'location': 'com.example.myapp:id/location_display',
            'contacts': 'com.example.myapp:id/contacts_list',
            'storage': 'com.example.myapp:id/file_selected'
        }

        element_id = verification_elements.get(permission_type)
        if element_id:
            try:
                success_element = self.driver.find_element(MobileBy.ID, element_id)
                assert success_element.is_displayed()
                return True
            except:
                return False

        return False

    def verify_permission_denied_handling(self, permission_type):
        """Verify app handles permission denial gracefully"""
        time.sleep(2)

        # Look for appropriate error message or alternative flow
        error_indicators = [
            'com.example.myapp:id/permission_error',
            'com.example.myapp:id/feature_unavailable',
            'com.example.myapp:id/permission_required_message'
        ]

        for indicator_id in error_indicators:
            try:
                error_element = self.driver.find_element(MobileBy.ID, indicator_id)
                if error_element.is_displayed():
                    return True
            except:
                continue

        return False

    def test_all_permissions(self):
        """Test all permission scenarios"""
        results = {}

        for permission_type in self.permission_scenarios.keys():
            try:
                # Reset app state
                self.driver.reset()

                # Test permission flow
                result = self.test_permission_request_flow(permission_type)
                results[permission_type] = result

            except Exception as e:
                results[permission_type] = f'error: {str(e)}'

        return results
```

### Network and Connectivity Testing
```python
# Network and connectivity testing
class NetworkTestSuite:

    def __init__(self, driver):
        self.driver = driver
        self.network_conditions = {
            'airplane_mode': 0,
            'wifi_only': 2,
            'data_only': 4,
            'wifi_and_data': 6
        }

    def test_network_scenarios(self):
        """Test app behavior under different network conditions"""
        results = {}

        for condition_name, condition_value in self.network_conditions.items():
            try:
                # Set network condition
                self.driver.set_network_connection(condition_value)
                time.sleep(3)

                # Test app functionality
                result = self.test_app_under_network_condition(condition_name)
                results[condition_name] = result

            except Exception as e:
                results[condition_name] = f'error: {str(e)}'

        # Restore normal connectivity
        self.driver.set_network_connection(6)

        return results

    def test_app_under_network_condition(self, condition):
        """Test app functionality under specific network condition"""
        test_results = {
            'app_responsive': False,
            'data_loads': False,
            'error_handling': False,
            'offline_features': False
        }

        try:
            # Test basic app responsiveness
            main_screen = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/main_screen')
            test_results['app_responsive'] = main_screen.is_displayed()

            # Test data loading
            if condition != 'airplane_mode':
                refresh_button = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/refresh_button')
                refresh_button.click()
                time.sleep(5)

                data_content = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/data_content')
                test_results['data_loads'] = data_content.is_displayed()

            # Test error handling for no connectivity
            if condition == 'airplane_mode':
                network_button = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/network_feature')
                network_button.click()
                time.sleep(3)

                error_message = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/network_error')
                test_results['error_handling'] = 'No connection' in error_message.text

                # Test offline features
                offline_button = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/offline_mode')
                offline_button.click()

                offline_content = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/offline_content')
                test_results['offline_features'] = offline_content.is_displayed()

        except Exception as e:
            print(f"Network test error for {condition}: {e}")

        return test_results

    def test_slow_network_performance(self):
        """Test app performance under slow network conditions"""
        # Simulate slow network (this requires network throttling tools)
        performance_metrics = {
            'page_load_time': 0,
            'image_load_time': 0,
            'api_response_time': 0,
            'user_experience_acceptable': False
        }

        try:
            # Test page load performance
            start_time = time.time()

            page_button = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/heavy_page_button')
            page_button.click()

            # Wait for page to load
            heavy_content = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/heavy_content')

            end_time = time.time()
            performance_metrics['page_load_time'] = end_time - start_time

            # Test image loading
            start_time = time.time()

            image_element = self.driver.find_element(MobileBy.ID, 'com.example.myapp:id/large_image')

            # Wait for image to load (check if image source is set)
            while not image_element.get_attribute('content-desc'):
                time.sleep(0.5)
                if time.time() - start_time > 30:  # Timeout after 30 seconds
                    break

            end_time = time.time()
            performance_metrics['image_load_time'] = end_time - start_time

            # Evaluate user experience
            total_load_time = performance_metrics['page_load_time'] + performance_metrics['image_load_time']
            performance_metrics['user_experience_acceptable'] = total_load_time < 10.0

        except Exception as e:
            print(f"Performance test error: {e}")

        return performance_metrics
```

## Resume Showcase Tips

### Strong Mobile App Testing Experience
```
✅ Excellent Examples:
• "Implemented comprehensive mobile testing strategy covering 25+ device types, reducing post-release defects by 70%"
• "Designed and executed automated Appium test suite for iOS/Android apps, improving test coverage by 85%"
• "Led cross-platform mobile testing initiative supporting React Native app across 50+ device configurations"
• "Developed mobile performance testing framework identifying and resolving 15+ critical user experience issues"

✅ Technical Skills to Highlight:
• Mobile automation: Appium, XCUITest, Espresso, UI Automator
• Device testing: Real devices, emulators, cloud device labs
• Platform expertise: iOS, Android, React Native, Flutter
• Mobile-specific testing: GPS, sensors, permissions, network scenarios
• Tools: BrowserStack, Sauce Labs, Firebase Test Lab, AWS Device Farm
```

### Quantifiable Achievements
- Device coverage improvements (number of devices/OS versions)
- Test automation coverage percentages
- Bug detection rates across different device types
- Performance optimization results
- User experience improvements measured through analytics

## Learning Path 📚

### Beginner (0-3 months)
1. **Mobile Testing Fundamentals**
   ```
   Core Concepts:
   - Mobile app types (native, hybrid, web)
   - iOS vs Android testing differences
   - Device emulators vs real devices
   - Mobile-specific testing challenges
   ```

2. **Manual Mobile Testing**
   - Touch gesture testing
   - Device orientation testing
   - Basic permission testing
   - App store compliance testing

3. **Tool Introduction**
   - Appium installation and setup
   - Android Studio and Xcode basics
   - Simple automated test creation

### Intermediate (3-8 months)
1. **Advanced Automation**
   ```python
   # Page Object Model for mobile
   class LoginPage:
       def __init__(self, driver):
           self.driver = driver

       @property
       def username_field(self):
           return self.driver.find_element(MobileBy.ID, 'username')

       def login(self, username, password):
           self.username_field.send_keys(username)
           # ... implementation
   ```

2. **Cross-Platform Testing**
   - Multi-device testing strategies
   - Cloud device lab usage
   - Performance testing on mobile
   - Security testing for mobile apps

3. **Specialized Testing**
   - GPS and location testing
   - Camera and media testing
   - Push notification testing
   - Background app testing

### Advanced (8+ months)
1. **Enterprise Mobile Testing**
   - Large-scale device management
   - CI/CD integration for mobile
   - Mobile DevOps practices
   - Advanced performance optimization

2. **Specialized Domains**
   - IoT and wearable testing
   - AR/VR application testing
   - Mobile game testing
   - Enterprise mobility management (EMM)

3. **Leadership and Strategy**
   - Mobile testing strategy development
   - Team leadership and mentoring
   - Tool evaluation and selection
   - Industry trend analysis

## Best Practices

### Test Strategy
```python
# Mobile test strategy implementation
class MobileTestStrategy:
    def __init__(self):
        self.test_pyramid = {
            'unit_tests': 70,  # 70% unit tests
            'integration_tests': 20,  # 20% integration tests
            'ui_tests': 10  # 10% UI tests
        }

        self.device_coverage = {
            'tier_1_devices': ['iPhone 13', 'Samsung Galaxy S21'],  # Latest popular
            'tier_2_devices': ['iPhone 11', 'Samsung Galaxy S20'],  # Previous gen
            'tier_3_devices': ['iPhone SE', 'Samsung Galaxy A50']   # Budget/older
        }

    def prioritize_tests(self):
        return [
            'Critical user journeys',
            'Platform-specific features',
            'Device-specific functionality',
            'Performance benchmarks',
            'Security validations'
        ]
```

### Continuous Testing
- Integrate mobile testing in CI/CD pipelines
- Automate device lab management
- Implement continuous performance monitoring
- Set up automated crash reporting analysis

### User Experience Focus
```python
def test_user_experience_metrics():
    """Test key UX metrics"""
    metrics = {
        'app_launch_time': measure_launch_time(),
        'screen_transition_smoothness': test_animations(),
        'touch_responsiveness': test_touch_latency(),
        'battery_impact': measure_battery_usage(),
        'memory_efficiency': monitor_memory_usage()
    }

    # Assert UX standards
    assert metrics['app_launch_time'] < 3.0  # seconds
    assert metrics['touch_responsiveness'] < 100  # milliseconds
    return metrics
```

---

**Next Step**: Test your data and databases! Continue to [Data & Database Testing](./10-data-database-testing.md) to learn ETL, migration, integrity, and SQL testing techniques.