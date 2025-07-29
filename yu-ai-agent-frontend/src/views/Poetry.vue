<template>
  <div class="poetry-container">
    <!-- 页面标题 -->
    <div class="header">
      <h1 class="title">小学生必背古诗75首</h1>
      <p class="subtitle">语文课程标准推荐篇目</p>
    </div>

    <!-- 搜索和筛选 -->
    <div class="search-section">
      <div class="search-box">
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="搜索诗名、作者或朝代..."
          class="search-input"
        />
        <button @click="clearSearch" class="clear-btn" v-if="searchQuery">清除</button>
      </div>
      
      <div class="filter-tabs">
        <button 
          v-for="dynasty in dynasties" 
          :key="dynasty"
          @click="selectedDynasty = dynasty"
          :class="['tab-btn', { active: selectedDynasty === dynasty }]"
        >
          {{ dynasty }}
        </button>
      </div>
    </div>

    <!-- 古诗列表 -->
    <div class="poetry-grid">
      <div 
        v-for="(poem, index) in filteredPoems" 
        :key="index"
        class="poem-card"
        @click="selectPoem(poem)"
        :class="{ selected: selectedPoem?.title === poem.title }"
      >
        <div class="poem-number">{{ poem.id }}</div>
        <h3 class="poem-title">{{ poem.title }}</h3>
        <p class="poem-author">{{ poem.dynasty }} · {{ poem.author }}</p>
        <div class="poem-preview">{{ poem.content.split('\n')[0] }}</div>
      </div>
    </div>

    <!-- 诗词详情弹窗 -->
    <div v-if="selectedPoem" class="poem-modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <button class="close-btn" @click="closeModal">×</button>
        <div class="poem-detail">
          <h2 class="detail-title">{{ selectedPoem.title }}</h2>
          <p class="detail-author">{{ selectedPoem.dynasty }} · {{ selectedPoem.author }}</p>
          <div class="detail-content">
            <p v-for="(line, index) in selectedPoem.content.split('\n')" :key="index" class="poem-line">
              {{ line }}
            </p>
          </div>
          <div class="poem-actions">
            <button @click="previousPoem" class="nav-btn" :disabled="currentIndex === 0">上一首</button>
            <button @click="nextPoem" class="nav-btn" :disabled="currentIndex === poems.length - 1">下一首</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// 响应式数据
const searchQuery = ref('')
const selectedDynasty = ref('全部')
const selectedPoem = ref(null)
const currentIndex = ref(0)

// 朝代列表
const dynasties = ['全部', '汉', '南北朝', '唐', '宋', '元', '明', '清']

// 75首古诗数据
const poems = ref([
  { id: 1, title: '江南', dynasty: '汉', author: '汉乐府', content: '江南可采莲，\n莲叶何田田。\n鱼戏莲叶间。\n鱼戏莲叶东，\n鱼戏莲叶西，\n鱼戏莲叶南，\n鱼戏莲叶北。' },
  { id: 2, title: '长歌行', dynasty: '汉', author: '汉乐府', content: '青青园中葵，朝露待日晞。\n阳春布德泽，万物生光辉。\n常恐秋节至，焜黄华叶衰。\n百川东到海，何时复西归？\n少壮不努力，老大徒伤悲。' },
  { id: 3, title: '敕勒歌', dynasty: '南北朝', author: '北朝民歌', content: '敕勒川，阴山下。\n天似穹庐，笼盖四野。\n天苍苍，野茫茫。\n风吹草低见牛羊。' },
  { id: 4, title: '咏鹅', dynasty: '唐', author: '骆宾王', content: '鹅，鹅，鹅，\n曲项向天歌。\n白毛浮绿水，\n红掌拨清波。' },
  { id: 5, title: '风', dynasty: '唐', author: '李峤', content: '解落三秋叶，\n能开二月花。\n过江千尺浪，\n入竹万竿斜。' },
  { id: 6, title: '咏柳', dynasty: '唐', author: '贺知章', content: '碧玉妆成一树高，\n万条垂下绿丝绦。\n不知细叶谁裁出，\n二月春风似剪刀。' },
  { id: 7, title: '回乡偶书', dynasty: '唐', author: '贺知章', content: '少小离家老大回，\n乡音无改鬓毛衰。\n儿童相见不相识，\n笑问客从何处来。' },
  { id: 8, title: '凉州词', dynasty: '唐', author: '王之涣', content: '黄河远上白云间，\n一片孤城万仞山。\n羌笛何须怨杨柳，\n春风不度玉门关。' },
  { id: 9, title: '登鹳雀楼', dynasty: '唐', author: '王之涣', content: '白日依山尽，\n黄河入海流。\n欲穷千里目，\n更上一层楼。' },
  { id: 10, title: '春晓', dynasty: '唐', author: '孟浩然', content: '春眠不觉晓，\n处处闻啼鸟。\n夜来风雨声，\n花落知多少。' },
  { id: 11, title: '凉州词', dynasty: '唐', author: '王翰', content: '葡萄美酒夜光杯，\n欲饮琵琶马上催。\n醉卧沙场君莫笑，\n古来征战几人回？' },
  { id: 12, title: '出塞', dynasty: '唐', author: '王昌龄', content: '秦时明月汉时关，\n万里长征人未还。\n但使龙城飞将在，\n不教胡马度阴山。' },
  { id: 13, title: '芙蓉楼送辛渐', dynasty: '唐', author: '王昌龄', content: '寒雨连江夜入吴，\n平明送客楚山孤。\n洛阳亲友如相问，\n一片冰心在玉壶。' },
  { id: 14, title: '鹿柴', dynasty: '唐', author: '王维', content: '空山不见人，\n但闻人语响。\n返景入深林，\n复照青苔上。' },
  { id: 15, title: '送元二使安西', dynasty: '唐', author: '王维', content: '渭城朝雨浥轻尘，\n客舍青青柳色新。\n劝君更尽一杯酒，\n西出阳关无故人。' },
  { id: 16, title: '九月九日忆山东兄弟', dynasty: '唐', author: '王维', content: '独在异乡为异客，\n每逢佳节倍思亲。\n遥知兄弟登高处，\n遍插茱萸少一人。' },
  { id: 17, title: '静夜思', dynasty: '唐', author: '李白', content: '床前明月光，\n疑是地上霜。\n举头望明月，\n低头思故乡。' },
  { id: 18, title: '古朗月行', dynasty: '唐', author: '李白', content: '小时不识月，\n呼作白玉盘。\n又疑瑶台镜，\n飞在青云端。' },
  { id: 19, title: '望庐山瀑布', dynasty: '唐', author: '李白', content: '日照香炉生紫烟，\n遥看瀑布挂前川。\n飞流直下三千尺，\n疑是银河落九天。' },
  { id: 20, title: '赠汪伦', dynasty: '唐', author: '李白', content: '李白乘舟将欲行，\n忽闻岸上踏歌声。\n桃花潭水深千尺，\n不及汪伦送我情。' },
  { id: 21, title: '黄鹤楼送孟浩然之广陵', dynasty: '唐', author: '李白', content: '故人西辞黄鹤楼，\n烟花三月下扬州。\n孤帆远影碧空尽，\n唯见长江天际流。' },
  { id: 22, title: '早发白帝城', dynasty: '唐', author: '李白', content: '朝辞白帝彩云间，\n千里江陵一日还。\n两岸猿声啼不住，\n轻舟已过万重山。' },
  { id: 23, title: '望天门山', dynasty: '唐', author: '李白', content: '天门中断楚江开，\n碧水东流至此回。\n两岸青山相对出，\n孤帆一片日边来。' },
  { id: 24, title: '别董大', dynasty: '唐', author: '高适', content: '千里黄云白日曛，\n北风吹雁雪纷纷。\n莫愁前路无知己，\n天下谁人不识君。' },
  { id: 25, title: '绝句', dynasty: '唐', author: '杜甫', content: '两个黄鹂鸣翠柳，\n一行白鹭上青天。\n窗含西岭千秋雪，\n门泊东吴万里船。' },
  { id: 26, title: '春夜喜雨', dynasty: '唐', author: '杜甫', content: '好雨知时节，\n当春乃发生。\n随风潜入夜，\n润物细无声。\n野径云俱黑，\n江船火独明。\n晓看红湿处，\n花重锦官城。' },
  { id: 27, title: '绝句', dynasty: '唐', author: '杜甫', content: '迟日江山丽，\n春风花草香。\n泥融飞燕子，\n沙暖睡鸳鸯。' },
  { id: 28, title: '江畔独步寻花', dynasty: '唐', author: '杜甫', content: '黄师塔前江水东，\n春光懒困倚微风。\n桃花一簇开无主，\n可爱深红爱浅红？' },
  { id: 29, title: '枫桥夜泊', dynasty: '唐', author: '张继', content: '月落乌啼霜满天，\n江枫渔火对愁眠。\n姑苏城外寒山寺，\n夜半钟声到客船。' },
  { id: 30, title: '滁州西涧', dynasty: '唐', author: '韦应物', content: '独怜幽草涧边生，\n上有黄鹂深树鸣。\n春潮带雨晚来急，\n野渡无人舟自横。' },
  { id: 31, title: '游子吟', dynasty: '唐', author: '孟郊', content: '慈母手中线，\n游子身上衣。\n临行密密缝，\n意恐迟迟归。\n谁言寸草心，\n报得三春晖。' },
  { id: 32, title: '早春呈水部张十八员外', dynasty: '唐', author: '韩愈', content: '天街小雨润如酥，\n草色遥看近却无。\n最是一年春好处，\n绝胜烟柳满皇都。' },
  { id: 33, title: '渔歌子', dynasty: '唐', author: '张志和', content: '西塞山前白鹭飞，\n桃花流水鳜鱼肥。\n青箬笠，绿蓑衣，\n斜风细雨不须归。' },
  { id: 34, title: '塞下曲', dynasty: '唐', author: '卢纶', content: '月黑雁飞高，\n单于夜遁逃。\n欲将轻骑逐，\n大雪满弓刀。' },
  { id: 35, title: '望洞庭', dynasty: '唐', author: '刘禹锡', content: '湖光秋月两相和，\n潭面无风镜未磨。\n遥望洞庭山水翠，\n白银盘里一青螺。' },
  { id: 36, title: '浪淘沙', dynasty: '唐', author: '刘禹锡', content: '九曲黄河万里沙，\n浪淘风簸自天涯。\n如今直上银河去，\n同到牵牛织女家。' },
  { id: 37, title: '赋得古原草送别', dynasty: '唐', author: '白居易', content: '离离原上草，\n一岁一枯荣。\n野火烧不尽，\n春风吹又生。\n远芳侵古道，\n晴翠接荒城。\n又送王孙去，\n萋萋满别情。' },
  { id: 38, title: '池上', dynasty: '唐', author: '白居易', content: '小娃撑小艇，\n偷采白莲回。\n不解藏踪迹，\n浮萍一道开。' },
  { id: 39, title: '忆江南', dynasty: '唐', author: '白居易', content: '江南好，\n风景旧曾谙。\n日出江花红胜火，\n春来江水绿如蓝。\n能不忆江南？' },
  { id: 40, title: '小儿垂钓', dynasty: '唐', author: '胡令能', content: '蓬头稚子学垂纶，\n侧坐莓苔草映身。\n路人借问遥招手，\n怕得鱼惊不应人。' },
  { id: 41, title: '悯农', dynasty: '唐', author: '李绅', content: '锄禾日当午，\n汗滴禾下土。\n谁知盘中餐，\n粒粒皆辛苦。' },
  { id: 42, title: '悯农', dynasty: '唐', author: '李绅', content: '春种一粒粟，\n秋收万颗子。\n四海无闲田，\n农夫犹饿死。' },
  { id: 43, title: '江雪', dynasty: '唐', author: '柳宗元', content: '千山鸟飞绝，\n万径人踪灭。\n孤舟蓑笠翁，\n独钓寒江雪。' },
  { id: 44, title: '寻隐者不遇', dynasty: '唐', author: '贾岛', content: '松下问童子，\n言师采药去。\n只在此山中，\n云深不知处。' },
  { id: 45, title: '山行', dynasty: '唐', author: '杜牧', content: '远上寒山石径斜，\n白云深处有人家。\n停车坐爱枫林晚，\n霜叶红于二月花。' },
  { id: 46, title: '清明', dynasty: '唐', author: '杜牧', content: '清明时节雨纷纷，\n路上行人欲断魂。\n借问酒家何处有？\n牧童遥指杏花村。' },
  { id: 47, title: '江南春', dynasty: '唐', author: '杜牧', content: '千里莺啼绿映红，\n水村山郭酒旗风。\n南朝四百八十寺，\n多少楼台烟雨中。' },
  { id: 48, title: '蜂', dynasty: '唐', author: '罗隐', content: '不论平地与山尖，\n无限风光尽被占。\n采得百花成蜜后，\n为谁辛苦为谁甜？' },
  { id: 49, title: '江上渔者', dynasty: '宋', author: '范仲淹', content: '江上往来人，\n但爱鲈鱼美。\n君看一叶舟，\n出没风波里。' },
  { id: 50, title: '元日', dynasty: '宋', author: '王安石', content: '爆竹声中一岁除，\n春风送暖入屠苏。\n千门万户曈曈日，\n总把新桃换旧符。' },
  { id: 51, title: '泊船瓜洲', dynasty: '宋', author: '王安石', content: '京口瓜洲一水间，\n钟山只隔数重山。\n春风又绿江南岸，\n明月何时照我还？' },
  { id: 52, title: '书湖阴先生壁', dynasty: '宋', author: '王安石', content: '茅檐长扫净无苔，\n花木成畦手自栽。\n一水护田将绿绕，\n两山排闼送青来。' },
  { id: 53, title: '六月二十七日望湖楼醉书', dynasty: '宋', author: '苏轼', content: '黑云翻墨未遮山，\n白雨跳珠乱入船。\n卷地风来忽吹散，\n望湖楼下水如天。' },
  { id: 54, title: '饮湖上初晴后雨', dynasty: '宋', author: '苏轼', content: '水光潋滟晴方好，\n山色空蒙雨亦奇。\n欲把西湖比西子，\n淡妆浓抹总相宜。' },
  { id: 55, title: '惠崇春江晓景', dynasty: '宋', author: '苏轼', content: '竹外桃花三两枝，\n春江水暖鸭先知。\n蒌蒿满地芦芽短，\n正是河豚欲上时。' },
  { id: 56, title: '题西林壁', dynasty: '宋', author: '苏轼', content: '横看成岭侧成峰，\n远近高低各不同。\n不识庐山真面目，\n只缘身在此山中。' },
  { id: 57, title: '夏日绝句', dynasty: '宋', author: '李清照', content: '生当作人杰，\n死亦为鬼雄。\n至今思项羽，\n不肯过江东。' },
  { id: 58, title: '三衢道中', dynasty: '宋', author: '曾几', content: '梅子黄时日日晴，\n小溪泛尽却山行。\n绿阴不减来时路，\n添得黄鹂四五声。' },
  { id: 59, title: '示儿', dynasty: '宋', author: '陆游', content: '死去元知万事空，\n但悲不见九州同。\n王师北定中原日，\n家祭无忘告乃翁。' },
  { id: 60, title: '秋夜将晓出篱门迎凉有感', dynasty: '宋', author: '陆游', content: '三万里河东入海，\n五千仞岳上摩天。\n遗民泪尽胡尘里，\n南望王师又一年。' },
  { id: 61, title: '四时田园杂兴', dynasty: '宋', author: '范成大', content: '昼出耘田夜绩麻，\n村庄儿女各当家。\n童孙未解供耕织，\n也傍桑阴学种瓜。' },
  { id: 62, title: '四时田园杂兴', dynasty: '宋', author: '范成大', content: '梅子金黄杏子肥，\n麦花雪白菜花稀。\n日长篱落无人过，\n惟有蜻蜓蛱蝶飞。' },
  { id: 63, title: '小池', dynasty: '宋', author: '杨万里', content: '泉眼无声惜细流，\n树阴照水爱晴柔。\n小荷才露尖尖角，\n早有蜻蜓立上头。' },
  { id: 64, title: '晓出净慈寺送林子方', dynasty: '宋', author: '杨万里', content: '毕竟西湖六月中，\n风光不与四时同。\n接天莲叶无穷碧，\n映日荷花别样红。' },
  { id: 65, title: '春日', dynasty: '宋', author: '朱熹', content: '胜日寻芳泗水滨，\n无边光景一时新。\n等闲识得东风面，\n万紫千红总是春。' },
  { id: 66, title: '观书有感', dynasty: '宋', author: '朱熹', content: '半亩方塘一鉴开，\n天光云影共徘徊。\n问渠那得清如许？\n为有源头活水来。' },
  { id: 67, title: '题临安邸', dynasty: '宋', author: '林升', content: '山外青山楼外楼，\n西湖歌舞几时休？\n暖风熏得游人醉，\n直把杭州作汴州。' },
  { id: 68, title: '游园不值', dynasty: '宋', author: '叶绍翁', content: '应怜屐齿印苍苔，\n小扣柴扉久不开。\n春色满园关不住，\n一枝红杏出墙来。' },
  { id: 69, title: '乡村四月', dynasty: '宋', author: '翁卷', content: '绿遍山原白满川，\n子规声里雨如烟。\n乡村四月闲人少，\n才了蚕桑又插田。' },
  { id: 70, title: '墨梅', dynasty: '元', author: '王冕', content: '我家洗砚池头树，\n朵朵花开淡墨痕。\n不要人夸好颜色，\n只留清气满乾坤。' },
  { id: 71, title: '石灰吟', dynasty: '明', author: '于谦', content: '千锤万凿出深山，\n烈火焚烧若等闲。\n粉骨碎身全不怕，\n要留清白在人间。' },
  { id: 72, title: '竹石', dynasty: '清', author: '郑燮', content: '咬定青山不放松，\n立根原在破岩中。\n千磨万击还坚劲，\n任尔东西南北风。' },
  { id: 73, title: '所见', dynasty: '清', author: '袁枚', content: '牧童骑黄牛，\n歌声振林樾。\n意欲捕鸣蝉，\n忽然闭口立。' },
  { id: 74, title: '村居', dynasty: '清', author: '高鼎', content: '草长莺飞二月天，\n拂堤杨柳醉春烟。\n儿童散学归来早，\n忙趁东风放纸鸢。' },
  { id: 75, title: '己亥杂诗', dynasty: '清', author: '龚自珍', content: '九州生气恃风雷，\n万马齐喑究可哀。\n我劝天公重抖擞，\n不拘一格降人才。' }
])

// 计算属性
const filteredPoems = computed(() => {
  let filtered = poems.value

  // 按朝代筛选
  if (selectedDynasty.value !== '全部') {
    filtered = filtered.filter(poem => poem.dynasty === selectedDynasty.value)
  }

  // 按搜索关键词筛选
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    filtered = filtered.filter(poem => 
      poem.title.toLowerCase().includes(query) ||
      poem.author.toLowerCase().includes(query) ||
      poem.dynasty.toLowerCase().includes(query) ||
      poem.content.toLowerCase().includes(query)
    )
  }

  return filtered
})

// 方法
const clearSearch = () => {
  searchQuery.value = ''
}

const selectPoem = (poem) => {
  selectedPoem.value = poem
  currentIndex.value = poems.value.findIndex(p => p.id === poem.id)
}

const closeModal = () => {
  selectedPoem.value = null
}

const previousPoem = () => {
  if (currentIndex.value > 0) {
    currentIndex.value--
    selectedPoem.value = poems.value[currentIndex.value]
  }
}

const nextPoem = () => {
  if (currentIndex.value < poems.value.length - 1) {
    currentIndex.value++
    selectedPoem.value = poems.value[currentIndex.value]
  }
}

// 键盘事件
onMounted(() => {
  document.addEventListener('keydown', (e) => {
    if (selectedPoem.value) {
      if (e.key === 'Escape') {
        closeModal()
      } else if (e.key === 'ArrowLeft') {
        previousPoem()
      } else if (e.key === 'ArrowRight') {
        nextPoem()
      }
    }
  })
})
</script>

<style scoped>
.poetry-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: 'Microsoft YaHei', sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 40px;
  color: white;
}

.title {
  font-size: 3rem;
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
  font-weight: bold;
}

.subtitle {
  font-size: 1.2rem;
  margin: 10px 0 0 0;
  opacity: 0.9;
}

.search-section {
  max-width: 800px;
  margin: 0 auto 40px auto;
}

.search-box {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.search-input {
  flex: 1;
  padding: 12px 20px;
  border: none;
  border-radius: 25px;
  font-size: 16px;
  background: rgba(255,255,255,0.9);
  backdrop-filter: blur(10px);
  outline: none;
  transition: all 0.3s ease;
}

.search-input:focus {
  background: rgba(255,255,255,1);
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.clear-btn {
  padding: 12px 20px;
  background: rgba(255,255,255,0.2);
  border: none;
  border-radius: 25px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.clear-btn:hover {
  background: rgba(255,255,255,0.3);
}

.filter-tabs {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

.tab-btn {
  padding: 8px 16px;
  border: 2px solid rgba(255,255,255,0.3);
  background: transparent;
  color: white;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
}

.tab-btn:hover {
  background: rgba(255,255,255,0.2);
}

.tab-btn.active {
  background: rgba(255,255,255,0.9);
  color: #667eea;
  border-color: rgba(255,255,255,0.9);
}

.poetry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.poem-card {
  background: rgba(255,255,255,0.95);
  border-radius: 15px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  position: relative;
  overflow: hidden;
}

.poem-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.poem-card.selected {
  border: 2px solid #667eea;
  box-shadow: 0 0 20px rgba(102, 126, 234, 0.3);
}

.poem-number {
  position: absolute;
  top: 10px;
  right: 15px;
  background: #667eea;
  color: white;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: bold;
}

.poem-title {
  font-size: 1.5rem;
  margin: 0 0 10px 0;
  color: #333;
  font-weight: bold;
}

.poem-author {
  color: #666;
  margin: 0 0 15px 0;
  font-size: 0.9rem;
}

.poem-preview {
  color: #555;
  font-style: italic;
  line-height: 1.6;
}

.poem-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
}

.modal-content {
  background: white;
  border-radius: 20px;
  padding: 40px;
  max-width: 600px;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
  margin: 20px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 20px;
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: #999;
  transition: color 0.3s ease;
}

.close-btn:hover {
  color: #333;
}

.poem-detail {
  text-align: center;
}

.detail-title {
  font-size: 2.5rem;
  margin: 0 0 15px 0;
  color: #333;
  font-weight: bold;
}

.detail-author {
  color: #666;
  margin: 0 0 30px 0;
  font-size: 1.1rem;
}

.detail-content {
  margin: 30px 0;
  line-height: 2;
}

.poem-line {
  font-size: 1.3rem;
  margin: 10px 0;
  color: #444;
  font-family: 'KaiTi', serif;
}

.poem-actions {
  display: flex;
  gap: 20px;
  justify-content: center;
  margin-top: 30px;
}

.nav-btn {
  padding: 12px 24px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 16px;
}

.nav-btn:hover:not(:disabled) {
  background: #5a6fd8;
  transform: translateY(-2px);
}

.nav-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .title {
    font-size: 2rem;
  }
  
  .poetry-grid {
    grid-template-columns: 1fr;
    gap: 15px;
  }
  
  .modal-content {
    margin: 10px;
    padding: 20px;
  }
  
  .detail-title {
    font-size: 2rem;
  }
  
  .poem-line {
    font-size: 1.1rem;
  }
  
  .filter-tabs {
    gap: 5px;
  }
  
  .tab-btn {
    padding: 6px 12px;
    font-size: 12px;
  }
}
</style>