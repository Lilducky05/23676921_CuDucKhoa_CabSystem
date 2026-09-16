openapi: 3.0.3
info:
  title: CAB System API - Booking Flow Updated
  description: Đặc tả API chi tiết cho luồng đặt xe dựa trên mô tả thực tế của quá trình điều phối.
  version: 1.1.0
servers:
  - url: https://api.cabsystem.com/v1
    description: Production Server
tags:
  - name: Customer
    description: Khách hàng thao tác đặt xe và theo dõi
  - name: Driver
    description: Tài xế nhận cuốc và cập nhật hành trình
paths:
  /bookings:
    post:
      summary: 1. Khách hàng tạo đơn đặt xe
      description: Khách hàng gửi điểm đón, điểm đến và loại xe. Hệ thống khởi tạo chuyến đi trạng thái SEARCHING.
      tags:
        - Customer
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/BookingRequest'
      responses:
        '201':
          description: Khởi tạo cuốc xe thành công, server bắt đầu rà quét xe gần nhất.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BookingResponse'

  /bookings/{tripId}/accept:
    post:
      summary: 2. Tài xế nhận cuốc xe
      description: Sau khi server bắn thông báo, tài xế đồng ý nhận chuyến. Hệ thống gán DriverID vào chuyến đi và đổi trạng thái thành ACCEPTED.
      tags:
        - Driver
      parameters:
        - name: tripId
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - driverId
              properties:
                driverId:
                  type: string
                  example: "DRV-10293"
      responses:
        '200':
          description: Nhận cuốc thành công. (Hệ thống ngầm bắn Push Notification báo cho khách hàng).
          
  /bookings/{tripId}/reject:
    post:
      summary: 2.1 (Ngoại lệ) Tài xế từ chối cuốc xe
      description: Tài xế bỏ qua cuốc xe. Hệ thống ghi nhận để tự động rà quét và chuyển cuốc cho xe ở gần tiếp theo.
      tags:
        - Driver
      parameters:
        - name: tripId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Từ chối thành công, server tiếp tục matching vòng lặp thứ 2.

  /drivers/locations:
    put:
      summary: 3. Cập nhật vị trí GPS liên tục của xe
      description: App tài xế liên tục ping tọa độ (mỗi 3-5s) để server cập nhật địa điểm xe hiện tại trên bản đồ.
      tags:
        - Driver
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DriverLocationPing'
      responses:
        '200':
          description: Đã ghi nhận tọa độ GPS.

  /bookings/{tripId}/status:
    get:
      summary: 4. Khách hàng nhận thông tin xe và ETA
      description: App khách hàng gọi API này để lấy thông tin tài xế, địa điểm xe hiện tại và thời gian bao nhiêu phút sẽ tới (ETA).
      tags:
        - Customer
      parameters:
        - name: tripId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Trạng thái realtime của chuyến đi và xe.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TripLiveStatus'

components:
  schemas:
    BookingRequest:
      type: object
      required:
        - customerId
        - pickupLat
        - pickupLng
        - dropoffLat
        - dropoffLng
        - serviceId
      properties:
        customerId:
          type: string
          example: "CUS-8849"
        pickupLat:
          type: number
          example: 10.762622
        pickupLng:
          type: number
          example: 106.660172
        dropoffLat:
          type: number
          example: 10.776889
        dropoffLng:
          type: number
          example: 106.700806
        serviceId:
          type: string
          example: "CAB-4"

    BookingResponse:
      type: object
      properties:
        tripId:
          type: string
          example: "TRIP-987654321"
        status:
          type: string
          example: SEARCHING
        fareEstimated:
          type: number
          example: 55000

    DriverLocationPing:
      type: object
      required:
        - driverId
        - currentLat
        - currentLng
      properties:
        driverId:
          type: string
          example: "DRV-10293"
        tripId:
          type: string
          description: Mã chuyến đang chạy (nếu có)
          example: "TRIP-987654321"
        currentLat:
          type: number
          example: 10.763000
        currentLng:
          type: number
          example: 106.661000

    TripLiveStatus:
      type: object
      properties:
        tripId:
          type: string
          example: "TRIP-987654321"
        status:
          type: string
          example: ACCEPTED
        etaMinutes:
          type: integer
          description: Thời gian bao nhiêu phút sẽ tới điểm đón
          example: 5
        driverInfo:
          type: object
          properties:
            fullName:
              type: string
              example: "Nguyen Van Tai Xe"
            licensePlate:
              type: string
              example: "51F-123.45"
            vehicleName:
              type: string
              example: "Toyota Vios - Trắng"
            currentLat:
              type: number
              example: 10.763000
            currentLng:
              type: number
              example: 106.661000
