# Sort multi-line arguments
* การแบ่งโค้ดให้เป็นหลายบรรทัด
* การทำแบบนี้จะช่วยให้หลีกเลี่ยงการพิมซ้ำ
* ง่ายต่อการที่จะอ่าน แก้ไข และปรับปรุง
* นอกจากนั้นยังจะทำให้คนที่จะกดยอมรับ pull request เข้าใจได้ง่ายและสามารถให้คำแนะนำได้อย่างถูกต้อง
* ให้พื้นที่ว่างก่อนที่จะใส่เครื่องหมาย \ ตัวอย่างเช่น
    #### GOOD
    * $docker container run -d --name mongo \\\
      --network demo-network  \\\
      -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \\\
      -e MONGO_INITDB_ROOT_PASSWORD=secret \\\
      -e MONGO_INITDB_DATABASE=mydb \\\
      mongo:4.2.8 

    #### BAD
    * $docker container run -d --name mongo --network demo-network -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=secret -e MONGO_INITDB_DATABASE=mydb mongo:4.2.8 

# Leverage build cache
* Docker จะทำตามคำสั่งใน **Dockerfile** ในขณะที่สร้าง Image เพื่อดำเนินการตามลำดับที่ระบุ เมื่อตรวจสอบคำสั่งแต่ละครั้ง Docker จะค้นหาภาพที่มีอยู่ในแคช (Caching) ที่สามารถนำมาใช้ซ้ำได้แทนที่จะสร้างภาพใหม่

* หากไม่ต้องการใช้แคช สามารถใช้ **--no-cache = true** บนคำสั่ง **docker build --no-cache = true**  อย่างไรก็ตามถ้าคุณปล่อยให้ Docker ใช้แคชมันเป็นสิ่งสำคัญที่จะต้องเข้าใจเมื่อมันสามารถและไม่สามารถหาภาพที่ตรงกันได้ตามกฎพื้นฐานที่ Docker มีดังต่อไปนี้:

  * เริ่มต้นด้วย **Parent image** ที่มีอยู่ในแคชแล้วหลังจากนั้นจะเปรียบเทียบกับ **Child image** ทั้งหมดที่มีในฐานข้อมูลนั้นเพื่อดูว่าภาพใดภาพหนึ่งถูกสร้างขึ้นโดยใช้คำสั่งเดียวกันไหม ถ้าไม่จะทำให้แคชไม่ทำงาน

  * ในกรณีส่วนใหญ่เพียงการเปรียบเทียบใน Dockerfile กับหนึ่งใน **Child image** ก็เพียงพอ

  * สำหรับคำสั่ง **ADD** และ **COPY** เนื้อหาของไฟล์ใน Image จะถูกตรวจสอบสำหรับแต่ละไฟล์ เวลาที่แก้ไขและเข้าถึงครั้งล่าสุดของไฟล์จะไม่ถูกพิจารณาในการตรวจสอบเหล่านี้ ในระหว่างการค้นหาแคชการตรวจสอบจะเปรียบเทียบกับการตรวจสอบใน Image ที่มีอยู่หรือเคยสร้างไปแล้ว หากมีการเปลี่ยนแปลงอะไรในไฟล์เช่นเนื้อหาและข้อมูลแสดงว่า แคชจะไม่ทำงาน

  * นอกเหนือจากคำสั่ง **ADD** และ **COPY** แล้วการตรวจสอบแคชไม่ได้ดูไฟล์ในคอนเทนเนอร์ ตัวอย่างเช่นเมื่อประมวลผลคำสั่ง **RUN apt-get -y update** ไฟล์ที่อัพเดตในคอนเทนเนอร์จะไม่ถูกตรวจสอบเพื่อพิจารณาว่ามีการเข้าใช้แคชหรือไม่ ในกรณีนั้นเพียงใช้สตริงคำสั่งเพื่อค้นหาข้อมูลที่ตรงกัน

* เมื่อแคชไม่ถูกต้องคำสั่ง Dockerfile ที่ตามมาทั้งหมดจะสร้างรูปภาพใหม่โดยไม่ใช้ **Caching**
![image](https://codefresh.io/wp-content/uploads/2017/03/Docker-cache-benchmark-1024x365.png)

# Dockerfile instructions
* คำแนะนำเหล่านี้ออกแบบมาเพื่อช่วยคุณสร้าง Dockerfile ที่มีประสิทธิภาพและบำรุงรักษาได้ง่าย
  ## FROM
  * คำสั่ง **FROM** เริ่มต้นการสร้างและการติดตั้ง **Base Image** ดังนั้น Dockerfile ที่ถูกต้องจะต้องเริ่มต้นด้วยคำสั่ง **FROM** การที่จะทำให้ **Image** เป็น **Image** ที่ถูกต้อง ควรที่จะเริ่มต้นโดยการดึง **Image** จาก **Public Repositories**
    * **FROM** สามารถใช้ได้หลายครั้งภายใน **Dockerfile** เดียวเพื่อสร้าง **Image** หรือใช้หนึ่งสเตจ **Build** เพื่อเป็นการอ้างอิงสำหรับอีก **Image** หนึ่ง เพียงแค่จดบันทึก **Image ID** ล่าสุดโดยการคอมมิตก่อนแต่ละคำสั่ง **FROM** อันใหม่ คำสั่ง **FROM** แต่ละครั้งจะล้างสเตจใด ๆ ที่สร้างขึ้นโดยคำสั่งก่อนหน้า
    * คุณสามารถตั้งชื่อให้กับสเตจสร้างใหม่โดยเพิ่ม **AS name** ไปยังคำสั่ง **FROM** โดยชื่อสามารถใช้ในภายหลังคำสั่ง **FROM** และ **COPY --from = <name | index>** เพื่ออ้างอิงรูปภาพที่สร้างขึ้นในสเตจนี้
    * **tag** หรือ **digest** เป็นทางเลือก หากคุณไม่เลือกตัวใดตัวหนึ่ง **Builder** จะใช้แท็ก **latest** เป็นค่าเริ่มต้น **Builder** จะส่งคืนข้อผิดพลาดหากไม่พบค่าแท็ก
  * **--platform** สามารถใช้ เพื่อระบุแพลตฟอร์มของ **Image** ในกรณีที่ **FROM** อ้างอิง **Image** แบบหลายแพลตฟอร์ม ตัวอย่างเช่น **linux/amd64, linux/arm64 หรือ windows/amd64** โดยค่าเริ่มต้นแพลตฟอร์มเป้าหมายของคำขอสร้างจะถูกใช้ ข้อโต้แย้งการสร้างระดับ Global สามารถนำมาใช้ในค่าของการตั้งค่าสเตจนี้ตัวอย่างเช่น automatic platform ARGs ช่วยให้สามารถบังคับให้สเตจสร้างแพลตฟอร์มแบบเนทีฟ (--platform = $ BUILDPLATFORM) และใช้เพื่อรวบรวมคอมไพล์ไปยังแพลตฟอร์มเป้าหมายภายในสเตจ
  * Example
    ```bash
    > FROM [--platform=<platform>] <image> [AS <name>]
    > FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
    ```

  ## LABEL
  * 
  ## RUN
  * 
