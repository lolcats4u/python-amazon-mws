############
Feeds
############
The Feeds API is used to submit changes to Amazon, and to manipulate the feeds that are submitted.The documentaiton for how to use the Feeds section of the API, what it does, and how Amazon handles the data can be found here: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_Overview.html

The python-amazon-mws wrapper currently supports the following Feeds API Functions.

& bull Submit Feed
& bull Get Feed Submission List
& bull Get Feed Submission List by Next Token
& bull Get Feed Submission Count
& bull Cancel Feed Submissions
& bull Get Feed Submission Result

Making a feeds_api Object
In order to use the Feeds API, Amazon need to know who you are, and which account to make changes to. This is done through your AmazonMWS account identifiers. You can find out how to fetch your account identifiers here:
https://sellercentral.amazon.com/forums/t/how-do-i-access-my-mws-access-key-and-secret-key/93944

Assuming you have already placed these items in your os environment, you need to make an object that submits the required informaiton, so that the information is handed to the Feeds API each time you use it, without the need of typing out the keys each time. I prefer to make my objects like this:

feeds_api=mws.Feeds(
        access_key=os.environ['MWS_ACCESS_KEY'],
        secret_key=os.environ['MWS_SECRET_KEY'],
        account_id=os.environ['MWS_ACCOUNT_ID'],
        region='US',
)

Once you are finished making this object, you can now use the Feeds API.

Submitting Feeds to Amazon
The goal of submitting a feed to Amazon is to give Amazon the information it needs to change something on your account. There are multiple different types of feeds. The list of the types of feeds and how they work can be found here:
http://docs.developer.amazonservices.com/en_US/feeds/Feeds_FeedType.html

In order to make a feed submission, you will need the following information in addition to the information needed to make a feeds_api object.

& bull The feed type and enumeration
& bull The feed itself

Feed Types and their Enumeration
Once you have selected the feed type you'd prefer to submit, you need to find out what the feed type enumeration is. This is a string of characters that Amazon uses to identify the type of feed you're submitting. Each feed type and their corresponding enumeration value, can be found using the feed types link above. For example, if I wanted to submit a change in inventory quantity, I would use the enumeration "_POST_INVENTORY_AVAILABILITY_DATA_"

Feed Types and their Standards
There are two ways to submit a feed to Amazon, xml, and csv(tabulated data) depending on the feed, they may accept one or both of these formats. 

In order to properly submit a feed to Amazon, these must be properly formatted. If you click the feed types link above, you will notice that there is a "file" seciton of the feed type table (This will require you to log in to you Amazon account). This gives you the informaiton you need to correctly format your data. For example, if I decide that I want to submit an inventory feed, and needed to know the format I would click on the Inventory.xsd file mentioned. This will give you an xml document that can be edited with the informaiton of your choice. Further examples can be found here:
https://sellercentral.amazon.com/gp/help/200386820
Inventory changes are often the first reason for someone to try to use the API, so there are many examples of how the feed works, its formatting, and how to properly make the xml feed.

Once you've constructed your feed, and placed your enumeration in your python program as a string, you can now submit your information to amazon. An example is provided below.

import mws
import os
import from bs4 BeautifulSoup

#this is the xml template provided in the updating quantity example in the Amazon MWS developer documentation. I have split it into headers, the template has been split into a header, the inner feed template, and end tags. This is because the header only has to be edited once, and can then be immediately appended to the message string. If you have to update multiple products you only need to copy the inner feed_template section, edit it, and append it to your messages list. Then you can add the end tags.

 header_text = """
    <?xml version="1.0" encoding="utf-8" ?>
    <AmazonEnvelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="amzn-envelope.xsd">
    <Header>
      <DocumentVersion>1.01</DocumentVersion>
      <MerchantIdentifier>M_SELLER_354577</MerchantIdentifier>
      </Header>"""
      
 feeds_template = """
      <MessageType>Inventory</MessageType>
    <Message>
      <MessageID>1</MessageID>
      <OperationType>Update</OperationType>
    <Inventory>
      <SKU>ASUSVNA1</SKU>
      <Quantity>8</Quantity>
      <FulfillmentLatency>1</FulfillmentLatency>"""
      
 end_tags"</Inventory>
      </Message>
    </AmazonEnvelope>
    """
#these need to be parsed so you can easily edit them using their pre assigned tags without having to do too much sting editing. The end tags do not have to be parsed, because they do not have to be edited.
    header = BeautifulSoup(header_text, "lxml")
    template = BeautifulSoup(feeds_template, "lxml")
    
#this is one of the main edits you need to make to your header. Amazon needs to know your merchant ID to submit it to the proper account inventory

    header.merchantidentifier.string = os.environ['MWS_ACCOUNT_ID']
    template.quantity = "0"
    
#Your now parsed and edited xml has be turned back into string to be added to the feed.
    header = str(header)
    template = str(template)
    
#now you can put together all of your inventory update feeds into a big list.
    message_string = []
    message_string = message_string.append(header)
    message_string = message_string.append(template)
    message_string = message_string.append(endtags)

#You can now join the message string with a new line to maintain readability
    message_string = '\n'.join(message_string)
    
#changed name just for clarity
    finished_feed = message_string
    
#amazon encodes their strings in utf-8 so you need to do this before you submit the feed
    feed = message_string.encode('utf-8')

#make sure you know your feed type enumeration
    feed_type = "_POST_INVENTORY_AVAILABILITY_DATA_"
    
#now using the feeds_api object you created earlier, you can submit the feed using the feed type enumeration listed.
    feeds_api.submit_feed(finished_feed, feed_type)
    
#ta da, you've updated your inventory!
    

