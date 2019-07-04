# Aliexpress Scraper and writing into csv File




from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
import random
import time
import csv


PROXY = "68.183.207.205:8080"
#198.211.108.93:8080
#PROXY = "150.242.21.27:8080"
options = Options()
options.add_argument('--start-maximized')
options.add_argument('--proxy-server=%s' % PROXY)
options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36')


driver = webdriver.Chrome(chrome_options=options)
main_page = ''
driver.get("https://www.aliexpress.com/?src=google&albch=fbrnd&acnt=304-410-9721&isdl=y&aff_short_key=UneMJZVf&albcp=54095668&albag=1897140028&slnk=&trgt=aud-165594907443:kwd-464198394164&plac=&crea=257006900048&netw=g&device=c&mtctp=e&memo1=1t1&albbt=Google_7_fbrnd&aff_platform=google&albagn=888888&gclid=CjwKCAjw0tHoBRBhEiwAvP1GFbYyQUsWbZAtaA0qywhTMRGNQ7fw5wxdHJBtZlbMou44XW0WVN2GfBoCJi0QAvD_BwE")



def resolve_modal_box():
    try:
        time.sleep(5)
        x_button = driver.find_element_by_xpath('/html/body/div[7]/div/div/a').click()
    except:
        pass
    try:
        time.sleep(5)
        x_button = driver.find_element_by_xpath('/html/body/div[6]/div/div/a').click()
    except:
        pass


#Function to get all the content of the selected link
def get_content():
    time.sleep(4)
    resolve_modal_box()
    sub_page = driver.window_handles[-1]
    print('sub page is : ')
    print(sub_page)
    time.sleep(10)
    windows = driver.window_handles
    print(windows)

    if len(windows) == 2:
        try:
            print("running block is window 2")
            time.sleep(5)
            driver.switch_to.window(sub_page)
            time.sleep(3)
            resolve_modal_box()
            time.sleep(4)
            price = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div[2]/div[4]/div[1]/span').text
            time.sleep(3)
            no_of_order = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div[2]/div[2]/span').text
            time.sleep(4)
            img_url = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div[1]/div/div/div[1]/img').get_attribute('src')
            time.sleep(10)
            driver.close()
            print('printing block 2 data')
            print('--------------------------------------')
            return dict({'img-url':img_url, 'price' : price, 'no-of-order' : no_of_order})
        except:
            driver.close()
            print('content cant be loaded')
    elif len(windows) == 3:
            print('--------------------------------------------')
            print("rusning block is running 3")
            for handle in windows:
                time.sleep(3)
                print('closing extra window')
                driver.switch_to.window(handle)
                if 'https://play.google.com/store/' in driver.current_url:
                    driver.close()
                    print('closing success')
                else:
                    pass
            time.sleep(2)
            sub_page = driver.window_handles[-1]
            try:
                time.sleep(5)
                driver.switch_to.window(sub_page)
                time.sleep(3)
                resolve_modal_box()
                time.sleep(4)
                price = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div[2]/div[4]/div[1]/span').text
                time.sleep(3)
                no_of_order = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div[2]/div[2]/span').text
                time.sleep(4)
                img_url = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div[1]/div/div/div[1]/img').get_attribute('src')
                time.sleep(10)
                driver.close()
                print("printing block 3 data")
                return dict({'img-url':img_url, 'price' : price, 'no-of-order' : no_of_order})
            except:
                driver.close()
                print('content cant be loaded')
    
def writing_into_csv(content):
    try:
        with open('aliexpress.csv','a') as csv_file:
           writer = csv.writer(csv_file)
           for key,value in content.items():
              writer.writerow([key,value])
    except:
        print("cant write this line in csv file")
        print("---------------------------------------------------")



#Selecting a single random main category
def all_categories():
    resolve_modal_box() 
    time.sleep(10)
    categories = driver.find_elements_by_css_selector('.cl-item > .cate-name > span > a')
    random_cate = random.randint(0,5)
    category = categories[random_cate]
    category.click()
    resolve_modal_box()
    time.sleep(3)
    return category

#Selecting a single random sub category
def sub_category():
    time.sleep(10)
    sub_categories = driver.find_elements_by_css_selector('.son-category > ul:nth-child(2) > li > a')
    end = len(sub_categories) - 1
    random_sub_cate = random.randint(0,end)
    sub_cate = sub_categories[random_sub_cate]
    time.sleep(7)
    sub_cate.click()
    time.sleep(7)
    resolve_modal_box()
    global main_page
    main_page = driver.current_window_handle
    print(main_page)
    return sub_cate


def getting_links():
    time.sleep(5)
    resolve_modal_box()
    time.sleep(8)
    links = driver.find_elements_by_css_selector('#list-items > ul > li > div > div.info > h3 > a')
    #getting the links from the subcategory page and scraping their data one by one
    for link in links:
        time.sleep(7)
        resolve_modal_box()
        time.sleep(3)
        driver.switch_to.window(main_page)
        time.sleep(5)
        link.click()
        resolve_modal_box()
        content = get_content()
        writing_into_csv(content)



    
    
    
    






main_category = all_categories()
sub_cate = sub_category()
getting_links()
            

