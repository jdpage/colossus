task :default => :check_links

task :check_links do
    pid = fork do
        exec 'jekyll serve'
    end
    sleep 3

    ok = true
    begin
        require 'anemone'
        root = 'http://localhost:4000/'
        puts "Checking links with anemone ... "
        # check-links --no-warnings http://localhost:4000
        Anemone.crawl(root, :discard_page_bodies => true) do |anemone|
            anemone.after_crawl do |pagestore|
                puts "killing jekyll server #{pid}"
                Process.kill("TERM", pid)

                broken_links = Hash.new { |h, k| h[k] = [] }
                pagestore.each_value do |page|
                    if page.code != 200
                        referrers = pagestore.pages_linking_to(page.url)
                        referrers.each do |referrer|
                            broken_links[referrer] << page
                        end
                    else
                        puts "OK #{page.url}"
                    end
                end
                puts "\n\nLinks with issues: "
                broken_links.each do |referrer, pages|
                    puts "#{referrer.url} contains the following broken links:"
                    pages.each do |page|
                        puts " HTTP #{page.code} #{page.url}"
                    end
                end
                ok = false if broken_links.length > 0
            end
        end
        puts "... done!"
    rescue LoadError
        abort 'Install anemone gem: gem install anemone'
    end
    fail "there were broken links" unless ok
end

