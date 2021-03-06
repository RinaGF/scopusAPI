# scopusAPI

The functions in this file allow you to query the Scopus Search API in R and parse the results into a data frame. The file contains three functions: searchByString() which allows you to query the API using an advanced search string, searchByID() which allows you to search for a list of article IDs (PMIDs, DOIs, or Scopus EIDs), and extractXML() which extracts values from the XML and parses them into a data frame you can work with in R.

## Before you begin

You will need to obtain a personal API key from Elsevier. You can request one at http://dev.elsevier.com/. Click on the "Get API Key" button, log in to the site, and then click on "register a new site." Enter the requested information and use your institution's home page as the site. 

You will then need to copy/paste your API key into the scopusAPI.R file at lines 9 and 52, replacing the yourAPIKey text with your API key.

You will also need to install the httr and XML packages, if you haven't already done so, with the command

    install.packages(c("httr", "XML"))

Finally, there are some API limits you should be aware of. 

First, if you want to download the full records of your search results (inlcuding the full author list and abstract for each document), you are limited to requesting 25 articles at a time. The searchByString() and searchByID() functions will automatically send multiple requests to the API to retrieve the full set of search results for your particular query, but it can only retrieve 25 full records at a time. 

Second, you can only request the first 5,000 records using the 'offset' parameter. This used to mean that you could only request up to 5,000 records for a single search string, but Elsevier has added a new 'cursor' parameter that allows you to bypass this limit. The new version of the searchByString() function uses this cursor parameter instead of the myStart parameter to iterate through the search results. In practice, this means you shouldn't use the myStart parameter unless you really need to. 

Finally, you are limited to downloading 20,000 records per week. 
 

## The searchByString() method

This function allows you to run an advanced search through the API and download all of the search results. I recommend developing the search string in the Scopus web interface and then using that string in the API to obtain the results. 

The function has eight arguments: string, content, myStart, retCount, retMax, mySort, cursor, and outfile. 
* **string:** the advanced search string you want to use.
* **content:** how many fields you want to return (either "complete" which returns all available fields or "standard" which returns an abbreviated record).
* **myStart:** which search result you want to start downloading from. Limited to the first 5,000 records for any given search string. Setting this value to 5,001 or higher will result in an error. In practice, you shouldn't need to use this unless your download process was interrupted and you want to pick up at the point where the error happened.
* **retCount:** how many records you want to download per request. Limited to 25 per request for "complete" content; requests for more than 25 with the "complete" content type will return an error.
* **retMax:** the maximum number of records you want to download. The function will continue to make requests until it reaches either the total number of search results or the retMax, if specified. If unspecified, it will return all of the search results. 
* **mySort:** how you want the search results to be sorted. Currently defaults to descending order by cover date, but could also be set to descending order by times cited count ("-citedby-count") or relevance ("-relevancy"). See the Scopus Search API wadl for more options. 
* **cursor:** a parameter used to iterate through a set of search results beyond 5,000. You shouldn't ever change this from it's default value. 
* **outfile:** the file you want to save the data to.

All but two of these arguments have default values: content defaults to "complete", myStart to 0, retCount to Inf, mySort to "-coverDate", cursor to the necessary value supplied by the Elsevier API, and retCount to 25. So, you only need to specify the string and the outfile for the function to work. 

## The searchByID() method

This function allows you to search for a list of article IDs and download the matching search results. It can search for PMIDs, DOIs, or EIDs (Scopus ID numbers). The function expects the list of article IDs to be either a character vector from R (e.g. myData$scopusID) or a text file with a single article ID per line.

The function has similar arguments and default values to the searchByString() method, but it also has an "idtype" argument which requires you to specify what kind of article ID you want to search for ("pmid", "doi", or "eid"). 

## The extractXML() function

This function extracts selected values from XML returned by either of the above methods and formats them into a data frame. Note, however, that this function will only work for XML returned using the "application/xml" datatype. If you want to work with json data instead of XML, I recommend parsing the results using the jsonlite package. 

## Sample workflow using the searchByString() method

Set your working directory and load the scopusAPI.R file

    setwd("C:/Users/Documents")
    source("scopusAPI.R")

Then save the search query you want to use. In this example I'm searching for documents that have the keyword "cryoelectron microscopy" and were published from 2006 to 2015.

    myQuery <- "KEY(\"cryoelectron microscopy\") AND PUBYEAR > 2005 AND PUBYEAR < 2016"

Next, run the search against the Scopus Search API and save the results in batches of 25 to a file called testdata.xml

    theXML <- searchByString(string = myQuery, outfile = "testdata.xml")
    
When the function finishes downloading all of the records, extract values from the XML and parse them into a data frame

    theData <- extractXML(theXML)

You can then work with the data in R or save it to a .csv file with the command

    write.csv(theData, file = "thedata.csv")

## Sample workflow using the searchByID() method

Set your working directory and load the scopusAPI.R file

    setwd("C:/Users/Documents")
    source("scopusAPI.R")

Then run the set of article IDs (in this example PMIDs) against the Scopus Search API and download the matching results 

    theXML <- searchByID(theIDs = "testPMIDs.txt", idtype = "pmid", outfile = "test.xml")

Then when the function is finished, extract the values from the resulting XML

    theData <- extractXML(theXML)

You can then work with the data frame in R or save it to a .csv, as above.
