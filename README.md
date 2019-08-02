<h2>Introduction</h2>
<hr />

<p>For four weeks of The Tech Academy, I worked on a team project developing personal and travel applications using Python and the Django framework. This was a great opportunity to get a real-world experience on a project with clear goals that needed to be met, and a limited amount of time in which to meet them. I learned what it is like to jump into the middle of a project and having to deal with bugs and utilizing or rewriting a previous programmer's code. I worked on stories involving back-end as well as front-end and gained vital experience working as part of a team.</p>
<hr />

<h3>Personal Application</h3>
<p>This application was intended for users to find movies, check the weather and traffic in a chosen area, and chat with other users.</p>
<hr />
The first story I took on was in the back-end of a page that would display information about the top five movies on IMDb. Although I laid the groundwork to include other information, the story I was working on only needed the names of the movies to be sent back to the view.

```

#Creating class for the movies. Just including title for now
class Movie(models.Model):
	title = models.CharField(max_length = 60)

	def __str__(self):
		return self.title
    
    
    # Download IMDB's Top 250 data
url = 'http://www.imdb.com/chart/top'
response = requests.get(url)
soup = BeautifulSoup(response.text, 'lxml')

#Get all necessary information from the IMDb webpage (title, link, cast)
movies = soup.select('td.titleColumn')
links = [a.attrs.get('href') for a in soup.select('td.titleColumn a')]
case = [a.attrs.get('title') for a in soup.select('td.titleColumn a')]
ratings = [b.attrs.get('data-value') for b in soup.select('td.posterColumn span[name=ir]')]
votes = [b.attrs.get('data-value') for b in soup.select('td.ratingColumn strong')]

#creating list to store movie titles


def topfive(request):
	
	imdb = []

	# Store each item into dictionary (data), then put those into a list (imdb)
	for index in range(0, 5):
		movie = Movie.objects.all()
		# get title
		movie_string = movies[index].get_text()
		movie = (' '.join(movie_string.split()).replace('.', ''))
		movie_title = movie[len(str(index))+1:-7]
		
		imdb.append(movie_title)

	context = {'imdb' : imdb}
		
	return render(request, 'movie/movie.html', context)
  ```
  
  My second story involved a mix of front-end and back-end work. I needed to create a way to display messages sent back and forth between users of the application like text messaging on a phone. I began to lay the ground work by creating a contacts list to store the different conversations based on which contact was sending the message.
  
  ```
  def conversations(request):
	contacts = []
	messages = Message.objects.filter(Q(sender=request.user) | Q(recipient=request.user)).order_by('-sent')
	for message in messages:
		if message.sender not in contacts:
			contacts.append(message.sender)
	return render(request, 'chat/conversations.html', {'contacts': contacts, 'messages': messages})
  
  ```
  
  Then I was creating the styling to show the different contacts in a sidebar that the user could select from to see the conversation. I was also contrasting the to and from messages. At this point, I had only gotten as far as showing the contrast based on nth-children and not on who sent the message, so it was just changing the color every other message. The next step would have been to distinguish between the user and the person they were contacting.
  
  HTML
```
<div class="wrapper">
    <div class="box header">Messages</div>
    <div class="box sidebar">
        <nav>
            <ul class="contactlist">
                {% for contact in contacts %}
                <li class="contact">{{ contact }}</li>
                {% endfor %}
            </ul>
        </nav>
    </div>
    <div class="box content">
        {% for message in messages %}
        <p>{{ message.messageBody }}</p>
        {% endfor %}
    </div>
</div>
```
  
  CSS
  ```
  <style>
    .sidebar {
        grid-area: sidebar;
        overflow: hidden;
        overflow-y: scroll;
    }

    .content {
        grid-area: content;
    }

    .header {
        grid-area: header;
    }


    .wrapper {
        display: grid;
        grid-gap: 10px;
        grid-template-columns: 200px 300px 120px;
        grid-template-areas: "....... header  header" "sidebar content content";
        color: #444;
    }

    .box {
        background-color: #ccc;
        color: #fff;
        border-radius: 5px;
        padding: 5px;
        font-size: 150%;
    }

    .header {
        background-color: #999;
    }

    nav ul.contactlist {
        padding-left: 0px;
        height: 200px;
        width: 100%;
        list-style-type: none;
    }

    li.contact {
        text-align: center;
        padding: 10px 0px;
    }

        li.contact:nth-child(even) {
            background: blue;
        }

        li.contact:nth-child(odd) {
            background: darkblue;
        }
</style>
```
<hr />

<h3>Travel Application</h3>
<p>This application was for users to find flights and hotels, set a budget, and make schedules for travel.</p>
<br />

My first task was to change the layout of the schedule page. There originally was a second navbar to select which page on the to-do list to go to, as well as add a new task. I got rid of the second navbar and set up a section to make it look better for creating new items on the to-do lists.

<h4>Before</h4>

![alt text](https://github.com/adavidsmith5/Python-Live-Projects/blob/master/before_todolist%20larger%20(1).png)

<h4>After</h4>

![alt text](https://github.com/adavidsmith5/pythonliveproject/blob/master/after_todolist.png)

<p>The next task I took on was changing how information from the airline API was displayed. Originally there was no airline name and the departure time and date was displayed in a format that was unreadable for most people.</p>

<h4>Custom Template Filters</h4>

```
amadeus = Client(
		client_id=...
		client_secret=...
	)
register = template.Library()

#Quick calculation to get the price of the flight before taxes.
@register.filter(name='pretax')
@stringfilter
def pretax(total,taxes):
	total = float(total)
	taxes = float(taxes)
	pretax = total - taxes
	return "% 12.2f" % pretax

#Changing the date and time from a string to datetime so that the correct format can be used
@register.filter(expects_localtime=True)
@stringfilter
def airlineName(airlineCode):
	response = amadeus.reference_data.airlines.get(airlineCodes=airlineCode)
	airline = response.data[0]['businessName']
	return airline

```

<h4>Calling the custom template filters in the view</h4>

```

{% for trip in dataAPI %} -----------------------------------------------------Flight(s) Trip Offer #{{ forloop.counter }} Start------------------------------------------------------------- {{ trip.offerItems.0.services.0.segments.1.flightSegment.arrival.at|date:'SHORT_DATE_FORMAT' }}
First Departure: {{ trip.offerItems.0.services.0.segments.0.flightSegment.departure.iataCode }}

Airline: {{ trip.offerItems.0.services.0.segments.0.flightSegment.carrierCode|airlineName }}

First Departure Terminal: {{ trip.offerItems.0.services.0.segments.0.flightSegment.departure.terminal }}

First Departure Date/Time: {{ trip.offerItems.0.services.0.segments.0.flightSegment.departure.at|dateformat|date:'N j, Y, P' }}

------------------
First Arrival: {{ trip.offerItems.0.services.0.segments.0.flightSegment.arrival.iataCode }}

First Arrival Terminal: {{ trip.offerItems.0.services.0.segments.0.flightSegment.arrival.terminal }}

First Arrival Time: {{ trip.offerItems.0.services.0.segments.0.flightSegment.arrival.at|dateformat|date:'N j, Y, P' }}

---Price---
<!--using a custom template filter to take the total price and subtract the taxes to find the pretax price. Custom filters are in the templatetags directory-->
<p><strong>Price:</strong> {{ trip.offerItems.0.price.total|pretax:trip.offerItems.0.price.totalTaxes }}</p>
<p><strong>Taxes:</strong> {{ trip.offerItems.0.price.totalTaxes }}

```

![alt text](https://github.com/adavidsmith5/Python-Live-Projects/blob/master/flight_info.png)
