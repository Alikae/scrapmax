require "resolv-replace"
require "open-uri"
require "nokogiri"
require "byebug"

def perform()
	datahash = {}
	actual = 0
	puts "Gemme an URL pls"
	city_names_url = import_city_urls(gets.chomp)
	city_names_url.each { |city_name, city_url|
		datahash[city_name] = get_mail(city_url)
		puts "importing (#{(100*actual/city_names_url.length).to_i+1}%):  " + datahash[city_name]
		actual += 1
	}
	return datahash
end

def write_in_txt(datahash)
	File.open("mails.txt", 'w') { |file|
		datahash.each { |name, mail|
			file.write("#{name}: \n ..........................................:#{mail} \n")
		}
	}
end

def import_city_urls(url)
	names_urls = {}
	doc = Nokogiri::HTML(open(url))
	doc.css('p a.lientxt').each { |link|
		names_urls[link["href"][5..-6]] = "http://annuaire-des-mairies.com" + link["href"][1..-1]
	}
	return names_urls
end

def get_mail(url)
	mail = "Pas de mail"
	begin
		doc = Nokogiri::HTML(open(url))
		doc.css('tr/td').each { |txt|
			if /@/ =~ txt.text
				mail = txt.text
			end
		}
	rescue StandardError => error
		mail = "error: #{error.class}"
	end
	return mail
end

def exec()
    write_in_txt(perform())
end

exec()
