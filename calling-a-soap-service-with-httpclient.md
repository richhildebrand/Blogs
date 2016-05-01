## Introduction

I recently took over a software department that is responsible for around two dozen of legacy applications. Most of these applications were written by junior developers with little to no oversight. 

A reoccurring theme has been that WCF services were used to send SOAP requests instead of creating JSON APIs. 

## The problem

This has cause several issue, but the most pressing of which has been that these calls do not work consistently across our development, QA, and production environments.

Prior to my arrival, this problem was solved by pointing some of our development applications at QA applications, and worse, by pointing some of our production applications at QA applications.

## The solution

Following the `clean as you go` software principle, each time we have enhancements or bug fixes for these legacy applications we are refactoring these SOAP calls to use the less fragile `HttpClient`.

This has allowed our API calls to work consistently across all of our environments.

## The code
For sending the request and extracted the response, we developed this helper class.


[code language="csharp"]

	namespace Infrastructure.ApiCalls.Helpers
	{
	    public static class SoapResponseHelper
	    {
	        public static T GetSoapResponse<T>(string webServiceUrl, string requestString)
	        {
	            var uri = new Uri(webServiceUrl);
	            var httpContent = new StringContent(requestString, Encoding.UTF8, "text/xml");
	
	            var response = FetchSoapResponse<T>(uri, httpContent);
	            return response;
	        }
	
	        private static T FetchSoapResponse<T>(Uri uri, HttpContent httpContent)
	        {
	            using (var client = new HttpClient())
	            {
	                var result = client.PostAsync(uri, httpContent).Result;
	                var resultContent = result.Content.ReadAsStringAsync().Result;
	
	                var responseContainer = DeserializeInnerSoapObject<T>(resultContent);
	                return responseContainer;
	            }
	        }
	
	        private static T DeserializeInnerSoapObject<T>(string soapResponse)
	        {
	            XmlDocument xmlDocument = new XmlDocument();
	            xmlDocument.LoadXml(soapResponse);
	
	            var soapBody = xmlDocument.GetElementsByTagName("soap:Body")[0];
	            string innerObject = soapBody.InnerXml;
	
	            return XmlHelper.Deserializer<T>(innerObject);
	        }
	    }
	}

[/code]

As well as this XML helper for deseralizing the extracted XML result.

[code language="csharp"]

	namespace Core.Helpers
	{
	    public static class XmlHelper
	    {
	        public static T Deserializer<T>(string objectAsString)
	        {
	            XmlSerializer deserializer = new XmlSerializer(typeof(T));
	
	            using (StringReader reader = new StringReader(objectAsString))
	            {
	                return (T)deserializer.Deserialize(reader);
	            }
	        }
	    }
	}

[/code]


## Summary
Switching to HttpClient has allowed us to fix these broken API calls without having to fix all of our applications at once.

As problems surface for our spaghetti environments, we will permanently fix these issues instead of participating in [plate emptying activities](http://agileotter.blogspot.com/2015/11/plate-emptying.html).