# pythonliveproject
Live Project for The Tech Academy after Python Course


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
  
  
  
  def conversations(request):
	contacts = []
	messages = Message.objects.filter(Q(sender=request.user) | Q(recipient=request.user)).order_by('-sent')
	for message in messages:
		if message.sender not in contacts:
			contacts.append(message.sender)
	return render(request, 'chat/conversations.html', {'contacts': contacts, 'messages': messages})
  
  
  
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
