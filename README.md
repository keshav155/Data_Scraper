# Data_Scraper_v2
## Project Scope
I decided to make this project in conjuction with SIT378, in which my client had actually requested me to develop a web scraping tool as a solution , which has free and removed the need to extract data manually. The goal of this project was to create a database which had the nutritional values of all food items in the top 10 fast food chain restaurants in Melbourne. The source from which I scraped the data out from was a free to access online data called [CalorieKing](https://www.calorieking.com/au/en/), and the solution was designed using 2 libraries in python namely:-

- [Beautifulsoup4](https://pypi.org/project/beautifulsoup4/)
- [Selenium](https://selenium-python.readthedocs.io/)

The solution was then further optimised with threading in Python with the help of [concurrent.future](https://docs.python.org/3/library/concurrent.futures.html) module, which is in-built in Python 3.

## Project Design and Implementation
The parallel project was designed while keeping in mind that concurrent.future is mainly used for speedening up IO-Bound tasks, since we are accessing and making requests a large amount of URL's (input), and then extracting information out of them (output). 

1. Preparing the Input (preparing name and URLS of each food item)
```python
def name_and_URL_fetcher(df , url):
    driver.get(url)
    host_website = 'https://www.calorieking.com'
    more_buttons = driver.find_elements_by_xpath("//a[@role = 'button']")

    for x in range(len(more_buttons)):
        if more_buttons[x].is_displayed():
            driver.execute_script("arguments[0].click();", more_buttons[x])
            time.sleep(1)
    page_source = driver.page_source
    
    soup = BeautifulSoup(page_source,'lxml')
    food_name = []
    links = []
    
    for url in soup.find_all("a",class_ = "MuiButtonBase-root MuiListItem-root MuiListItem-gutters MuiListItem-button",href=True):
        links.append(url['href'])
    
    for name in soup.find_all("span",class_ = "MuiTypography-root MuiListItemText-primary jss372 MuiTypography-body1 MuiTypography-noWrap MuiTypography-displayBlock"):
        food_name.append(name.text)
        
    final_urls = [host_website + x for x in links]    
    df['Name'] = food_name
    df['URL'] = final_urls
```
2. Fetching the output (extracting nutritional value of each food item)
```python
def data_fetcher_v2(url):
            try: # food item
                my_url = url

                source = requests.get(my_url).text

                soup = BeautifulSoup(source,'lxml')

                calories = soup.find('th',class_="MuiTableCell-root MuiTableCell-body jss465 MuiTableCell-sizeSmall").text
                calories = calories.split()
                calories = calories[1] + " " +calories[0] 
                #print(calories)

                kilojoule = soup.find('td',class_="MuiTableCell-root MuiTableCell-body MuiTableCell-alignRight MuiTableCell-sizeSmall").text
                kilojoule = kilojoule[1:-1]
                #print(kilojoule)

                protein = soup.find('th',text = 'Protein').parent.td.text
                #print(protein)

                total_fats = soup.find('th',text = 'Total Fat').parent.td.text
                #print(total_fats)

                saturated_fats = soup.find('th',text = 'Saturated Fat').parent.td.text
                #print(saturated_fats)

                carbohydrates = soup.find('th',text = 'Carbohydrate').parent.td.text
                #print(carbohydrates)

                sugars = soup.find('th',text = 'Sugars').parent.td.text
                #print(sugars)

                sodium = soup.find('th',text = 'Sodium').parent.td.text
                data_to_insert = [ calories,kilojoule,protein,total_fats,saturated_fats,carbohydrates,sugars,sodium ]
             #   data_to_be_inserted_parallel.append(data_to_insert)
                print("Data has been scraped successfully")
                return data_to_insert
    #            print(str(extract_data) + " has been scraped successfully")
            except: # meal
                my_url = url

                source = requests.get(my_url).text

                soup = BeautifulSoup(source,'lxml')

                calories = soup.find('th',class_="MuiTableCell-root MuiTableCell-body jss466 MuiTableCell-sizeSmall").text
                calories = calories.split()
                calories = calories[1] + " " +calories[0] 
                #print(calories)

                kilojoule = soup.find('td',class_="MuiTableCell-root MuiTableCell-body MuiTableCell-alignRight MuiTableCell-sizeSmall").text
                kilojoule = kilojoule[1:-1]
                #print(kilojoule)

                protein = soup.find('th',text = 'Protein').parent.td.text
                #print(protein)

                total_fats = soup.find('th',text = 'Total Fat').parent.td.text
                #print(total_fats)

                saturated_fats = soup.find('th',text = 'Saturated Fat').parent.td.text
                #print(saturated_fats)

                carbohydrates = soup.find('th',text = 'Carbohydrate').parent.td.text
                #print(carbohydrates)

                sugars = soup.find('th',text = 'Sugars').parent.td.text
                #print(sugars)

                sodium = soup.find('th',text = 'Sodium').parent.td.text
                #print(sodium)
                data_to_insert = [ calories,kilojoule,protein,total_fats,saturated_fats,carbohydrates,sugars,sodium ]
          #      data_to_be_inserted_parallel.append(data_to_insert)
                print("Data has been scraped successfully")
                return data_to_insert
```





3. Run data_fetcher_v2 prepared in step 2 on the URLS's prepared in step 1.
This is where all the magic happens:-

```python
t1 = time.perf_counter()
with concurrent.futures.ThreadPoolExecutor() as executor:
    data_to_be_inserted_parallel = executor.map(data_fetcher_v2, restaurant_url_list)
t2 = time.perf_counter()
print(f'Finished in {t2-t1} seconds')
```

Let’s take a look at how this code works:-

- The `with` statement is used to create a `ThreadPoolExecutor` instance executor that will promptly clean up threads upon completion, hence we do not have to manually close any threads
- `data_fetcher_v2` returns the macronutrientional information of a single food item.
- `executor.map()` gives results in the order they are submitted, hence, not requiring us to use mutex lock
- `restaurant_url_list` is passed in , which is a list of all food item URL's extracted previously, instead of looping through each restaurant URL one by one (`restaurant_df['URL'][0] ,restaurant_df['URL'][1],restaurant_df['URL'][2] ...`) 

## Project Evaluation

Here are the timings and evaluation of the [top 10 fast food chains restaurants](https://www.statista.com/statistics/871574/australia-leading-fast-food-restaurant-chains/) as per statista in 2018.
| Restaurant        |Number of food items| Sequential (seconds)          | Parallel (seconds) | 
| :---------------: |:------------------:|:--------------------:| :--------:|
| Mcdonalds         |207| 389.52        | 20.78     |
| KFC               |88| 96.39            |   5.56     |
| Subway            |107| 149.94             |    7.88     |
| Hungry Jacks      |207| 380.93            |    18.09     |
| Domino's Pizza    |106| 190.74            |    8.65     |
| Red Rooster       |76| 115.23           |    5.94     |
| Grill'd           |42| 92.76             |    4.73     |
| Nandos            |64| 100.80            |    5.18     |
| Pizza Hut         |83| 120.48           |    7.70     |
| Noodle Box        |25| 59.69           |    3.97     |
| <b>Total</b>       |1005|<b> 1693.48   </b>       |   <b> 88.48  </b>   |

## Conclusion
I was able to save myself 1605 seconds, which is 26.75 minutes, and in the future, if the database gets updated, I would be easily able to update my excel sheets again in 1.47 minutes, which is around <b>20 times</b> faster if I were to use the sequential solution.

## Acknowledgements

Special thanks to [Cedric](https://github.com/Coder-3), another student of SIT315 on introducing me to web-scraping with python
