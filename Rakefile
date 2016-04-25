require "stringex"
require 'json'
require 'qiniu'

posts_dir       = "_posts"    # directory for blog files
new_post_ext    = "markdown"  # default new post file extension when using the new_post task

# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  mkdir_p "#{posts_dir}"
  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: #{title.gsub(/&/,'&amp;')}"
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M:%S %z')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
  end
end

#提交代码到github
desc "make new file"
task :deploy do
  system "rake uploadFile[#{posts_dir}]"
  system "git add ."
  system "git commit -m \"deploy blog\""
  system "git push"
end

#上传文件到七牛
desc "upload image to qiniu"
task :uploadFile, :file_dir do |t, args|
  # File.open(args.file_dir,"r") do |file|  
  #   while line = file.gets   #标准输入流  
  #      puts line  
  #   end  
  #  end 
  file_dir = args.file_dir
  modfiles = get_mod_files file_dir,".markdown"
  if modfiles.length == 0
    puts "no file modified"
    next
  end
  scan_img_path file_dir, modfiles
end

#扫描文件，获取图片路径
desc "scan image path contained in file"
def scan_img_path(file_dir, files)
  files.each do |one_file|
    file_content = File.read("#{file_dir}/#{one_file}")
    matchs = file_content.scan(/^!\[image\]\(([^(http)]+[^\(^\)]*)\)/)
    puts matchs
    if matchs.length == 0
      save_mod_time file_dir, one_file
      next
    end

    matchs.each do |image_path| 
      _path = image_path[0].to_s
      _path = _path.sub(/(..\/)/,'')
      code = upload_file_to_qiniu _path
      if code == 200
        save_mod_time file_dir, one_file
      end  
    end
  end
end

#保存当前文件的修改时间
def save_mod_time(file_dir, file)
  mtime = File.mtime("#{file_dir}/#{file}").tv_sec

  #文件修改时间的 hash 表
  json = File.read("#{file_dir}/.mdy")
  if json.length > 0
    hashMap = JSON.parse(json)  
  else
    hashMap = Hash.new("0")  
  end

  hashMap["#{file}"] = "#{mtime}"

  mdy_file = File.open("#{file_dir}/.mdy","w")
  mdy_file.puts hashMap.to_json
  mdy_file.close

end

################################
# modified markdown file  
################################

#获取路径下的给定后缀的文件
def get_files_with_ex(file_dir, file_extra)
  #files array
  files = Array.new

  #遍历路径下面的文件，获取匹配 file_extra 的文件
  Dir.foreach(file_dir) {|file| 
    extra = File.extname(file)
    if extra == file_extra
      files.push(file)
    end
  }
  return files
end

#获取 file_dir 路径下， file_extra 为后缀的文件，被修改过的
def get_mod_files(file_dir, file_extra)

  #修改过的文件
  mod_file = Array.new

  #获取 file_dir 路径下， file_extra 为后缀的文件
  files = get_files_with_ex file_dir, file_extra

  #文件修改时间的 hash 表
  json = File.read("#{file_dir}/.mdy")
  should_init_mdy = 0
  if json.length > 0
    hashMap = JSON.parse(json.to_s)
  else
    hashMap = Hash.new("0")  
    should_init_mdy = 1
  end

  hashMap = Hash.new("0")  
  
  files.each do |file_name|
    mtime = File.mtime(file_dir+'/'+file_name).tv_sec
    new_mtime = mtime.to_i
    old_mtime = hashMap[file_name].to_i
    if new_mtime > old_mtime
        mod_file.push(file_name)
    end
    if should_init_mdy == 1
      hashMap[file_name] = new_mtime.to_s
    end
  end

  if should_init_mdy == 1
    save_mdy = File.open("#{file_dir}/.mdy", "w")
    save_mdy.puts hashMap.to_json
    save_mdy.close
  end

  return mod_file
end


######################
# 七牛上传
#####################

def upload_file_to_qiniu(file_path)
  # 构建鉴权对象
  Qiniu.establish_connection! :access_key => 'wzqrJ1ftbHzYaNRFT4z5akcw2Xt0Ud-wCjCLa6B2',
                              :secret_key => 'TAvWSsUWzCx4Z0hkBo2RI6zqPaimSCmIYI7n2Cfu'

  #要上传的空间
  bucket = 'gukong-blog-images'

  #上传到七牛后保存的文件名
  file_name = file_path.match(/([^\/]*)(.png|.jpg)/).to_s
  if file_name.length == 0
    file_name = Time.now.strftime("%Y-%m-%d-%H:%M:%S")
  end
  key = "image/" + file_name

  #构建上传策略
  put_policy = Qiniu::Auth::PutPolicy.new(
      bucket,      # 存储空间
      key,     # 最终资源名，可省略，即缺省为“创建”语义，设置为nil为普通上传 
      3600    #token过期时间，默认为3600s
  )

  #生成上传 Token
  uptoken = Qiniu::Auth.generate_uptoken(put_policy)

  #调用upload_with_token_2方法上传
  code, result, response_headers = Qiniu::Storage.upload_with_token_2(
       uptoken, 
       file_path,
       key
  )

  #打印上传返回的信息
  return code
end

